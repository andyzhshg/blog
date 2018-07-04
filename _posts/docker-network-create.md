title: 用network create解决Docker容器互相连接的问题
date: 2016-10-09 20:00:00
tags: [技术, docker]
categories: 泛技术

---

目前在做一个P2P的程序，为了方便测试和部署，我们用`docker`来运行相同的程序实例。

作为一个P2P的程序，所有的实例之间应该是对等的，并且可以相互连接通信，看似简单，一开始的时候我们却在这里栽了跟头...

<!-- more -->

通常情况下，我们是通过在启动容器的时候传入`--link`参数的方式访问其他的容器的。

比如我们的一个容器要访问`mysql`，我们这样启动`mysql`(注意`--name`参数)：

```
docker run -e MYSQL_ROOT_PASSWORD=password --name=mysql -d mysql
```

然后我们的需要访问`mysql`的容器这样启动(注意`--link`参数)

```
docker run --name=my_container --link mysql:mysql my/my_image
```

这样，在我们新启动的容器`my_container`里就能够以域名`mysql`来访问`mysql`容器了。

可现在的情况是，作为一个P2P的程序，我们要启动两个容器(至少两个)`container_a`和`container_b`。希望`container_a`能访问`container_b`，并且`container_b`可以访问`container_a`，用跟上边同样的思路我们会想到这样启动

```
docker run --name=container_a --link container_b:container_b -d my/my_image
docker run --name=container_b --link container_a:container_a -d my/my_image
```

但是问题来了，当我们执行第一句命令时，`docker`会告诉我们，`container_b`不存在，可不是么，`container_b`此时还没有启动，当然是不存在的。这就造成了一种困境，`container_a`和`container_b`两者是相互依赖的，二者无论先启动谁，都要求先启动另一者，这种方式根本解决不了这个问题。

那么这种容器互相连接的问题怎么解决呢？就要轮到本文重点`docker network create`登场了。

首先我们先创建一个`network`

```
docker network create my-net
```

通过执行命令可以查看到我们创建的网络

```
docker network ls
```

结果应该类似下面这样

```
cbd1aa22d04a        bridge              bridge              local
33be010f6cd9        host                host                local
628c2540d5d3        my-net              bridge              local
```

我们看到`my-net`在这个列表中，然后我们就要利用这个新建的`my-net`网络来启动我们的容器。

```
docker run --net=my-net --net-alias=container_a --name=container_a -d my/my_image
docker run --net=my-net --net-alias=container_b --name=container_b -d my/my_image
```

我们通过`--net`参数指定容器使用的网络，通过`--net-alias`指定容器在这个网络中的别名，这样在这个网络中的所有的容器就都可以通过这个别名作为域名来访问到该容器了。











