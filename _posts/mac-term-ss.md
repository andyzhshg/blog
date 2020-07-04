title: mac的终端用ss做代理
date: 2019-02-15 16:05:00
tags: [技术, git, ss,mac,go]
categories: 泛技术

------

最近在研究`golang`，`go`提供了一个很好用的工具就是`go get`。`go get`本身不得不说是一个伟大的设计，极大地减轻了我们做包管理的负担，这个对于主要做`C++`的我的感受尤为明显。

但是在我们这个国家里，因为某些你懂得原因，顺利的用`go get`却成了一件很困难的事。之前写过一篇文章介绍[如何让`git`走`SS`代理](http://www.up4dev.com/2018/05/14/github-use-ss/)，但是`go get`还是与`git`的情况不是完全一致的。

`google`了一圈之后，我总结了一个更加通用的方法。

<!--more-->

理论上这个方法可以让所有的终端的命令通过`SS`提供的`socks5`代理来访问网络。

方法很简单，就是在需要用代理的时候运行如下的命令：

```bash
export all_proxy=socks5://127.0.0.1:1080
```

其中`socks5://127.0.0.1:1080`是`SS`提供的`socks5`代理服务的监听地址，可以在`Shadowsocks`的高级设置下找到。

如果想减少每次打开执行导入命令的麻烦，可以将导入语句加入到`~/.bash_profile`中。



------

#### 参考：

https://github.com/mrdulin/blog/issues/18

https://studygolang.com/articles/9490

https://github.com/golang/go/wiki/GoGetProxyConfig

