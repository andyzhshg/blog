title: 一个用 etcd 作主备锁的细节问题
date: 2019-05-29 16:04:00
tags: [etcd, go, 技术,分布式,锁]
categories: 技术

------

`etcd` 越来越多的被应用在分布式系统中，最典型的一个应用场景就是作为分布式锁，用于在分布式系统中保证资源的独占。

通常的分布式的锁的应用场景有如下几种方式：

1. 即用即申请，用完即释放，一般用于资源控制粒度比较细的系统，这种场景会频繁的调用 `etcd` 服务
2. 还有一种就是先到先得，得到即长期占有，这种更多是用在主备系统的切换场景，如果占有锁的服务不发生异常，则不会主动与 `etcd` 交互。

本文主要讨论第2种场景中遇到的一个细节问题。

<!-- more -->

在主备切换的场景，我们希望服务一旦获取到锁，就不必主动的与 `etcd` 交互，而是专心的进行自己的本职工作。但是如果不主动跟 `etcd` 询问持有的锁的状态的话，我们又无法保证当前是确实持有锁的。正如下边的代码：

```go
package main

import (
	"context"
	"fmt"
	"time"

	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/clientv3/concurrency"
)

func main() {
	client, errClient := clientv3.New(clientv3.Config{Endpoints: []string{"http://127.0.0.1:2379"}, DialTimeout: 10 * time.Second})
	if errClient != nil {
		fmt.Errorf("client create fail - %v", errClient)
		return
	}
	session, errSession := concurrency.NewSession(client, concurrency.WithTTL(10))
	if errSession != nil {
		fmt.Errorf("create session fail - %v", errSession)
		return
	}

	mutex := concurrency.NewMutex(session, "/lock")
	if mutex == nil {
		fmt.Errorf("create mutex fail")
		return
	}
	errMutex := mutex.Lock(context.TODO())
	if errMutex != nil {
		fmt.Errorf("lock fail - %v", errMutex)
		return
	}

	fmt.Println("got lock, begin run work")

	go func() {
		// do real work here
	}()

	// prevent progress quit
	select {}
}

```

在 `// do real work here` 执行过程中，很可能我们的网络状态出现了问题，或者 `etcd` 服务出现问题导致程序已经跟网络断开，这时实际上锁很可能已经失效了。为了保证锁的有效性，我们可以在`session`的有效期内轮询锁的状态，但是这种做法很繁琐，也比较浪费资源。有没有更好的方式呢？

好在 `session` 提供了一个 `Done` 方法，该方法返回一个 `channel` ， 一旦 `session` 结束，这个 `channel` 就会被写入内容，这样就给了我们一个简单地方法来监控锁的状态。

```go
package main

import (
	"context"
	"fmt"
	"time"

	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/clientv3/concurrency"
)

func main() {
	client, errClient := clientv3.New(clientv3.Config{Endpoints: []string{"http://127.0.0.1:2379"}, DialTimeout: 10 * time.Second})
	if errClient != nil {
		fmt.Errorf("client create fail - %v", errClient)
		return
	}
	session, errSession := concurrency.NewSession(client, concurrency.WithTTL(10))
	if errSession != nil {
		fmt.Errorf("create session fail - %v", errSession)
		return
	}

	mutex := concurrency.NewMutex(session, "/lock")
	if mutex == nil {
		fmt.Errorf("create mutex fail")
		return
	}
	errMutex := mutex.Lock(context.TODO())
	if errMutex != nil {
		fmt.Errorf("lock fail - %v", errMutex)
		return
	}

	fmt.Println("got lock, begin run work")

	go func() {
		select {
		case <-session.Done():
			// do what ever you want to process lock lost
			fmt.Println("lock lost")
		}
	}()

	go func() {
		// do real work
	}()

	// prevent progress quit
	select {}
}
```

如上边的代码所示，我们在一个 `goroutine` 中监听一个 `<-session.Done()` 的 `channel` ，这样，一旦锁出现了问题，就会得到通知，这样就可以在这里进行一些锁丢失的善后工作，比如在这里停止所有的需要锁才能进行的工作，这样就不会出现锁已经失效，但是工作进程却全然不知的状况了。