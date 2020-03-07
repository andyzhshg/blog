title: 用 Docker 为 Trojan 客户端添加 HTTP 代理
date: 2020-03-07 15:16:00
tags: [技术,Docker,VPN]
categories: 泛技术

---

trojan 是近期热度很大的一个代理软件，本文不打算教大家怎么搭建 torjan，而是描述一下我如何解决使用 trojan 的时候遇到的一些问题。

我使用 trojan 的主要目的不是为了翻越那个伟大的长城，我笃信专业的事情找专业的人来干，花钱买机场比自己搭建来说省时省力。使用 trojan 主要是为了能够通过它提供的代理隧道来使用 openvpn。OpenVPN Connect 的客户端只支持 HTTP 代理，不支持 socks5，而 trojan 默认只支持 socks5 代理，所以必须做一层转换，将 socks5 代理协议转换成 HTTP 代理协议。

<!-- more -->

我用的办法是 privoxy，而为了使用的方便，我将 trojan 的客户端和 privoxy 打包进了 docker 镜像中。好处就是无论在什么环境，只要是执行一句 `docker run` 指令就可快速的拉起一个既支持 socks5 又支持 HTTP 协议的代理客户端。

而且现在这种新冠疫情下，大部分时间都是在家工作，家里大大小小的设备一堆，给每个设备都部署 trojan 非常麻烦。所以用 docker 在家里的 NAS 上部署一个 trojan 然后所有设备都共享这个代理，就方便了很多。

实现的方法很简单，我是在一个 ubuntu 的基础镜像之上安装了 privoxy 来提供 HTTP 代理服务，然后从 github 官方项目下载 最新版的 trojan 客户端。提供一个脚本来启动这两个应用，而在脚本中提供了可以通过环境变量来设置 trojan 的基本配置参数。

使用的方法很简单，可以通过下面的命令来快速的启动：

```bash
docker run \
    --name trojan-proxy \
    -e REMOTE_ADDR="your host" \
    -e PASSWORD="your password" \
    -p trojan_port:1086 \
    -p privoxy_port:1087 \
    -d \
    andyzhshg/trojan-privoxy:latest
```

其中参数环境变量 `REMOTE_ADDR` 用于指定 trojan 的服务地址，而 `PASSWORD` 用于指定服务的密码。

1086 端口是 socks5 代理的端口， 1087 是 HTTP 代理的端口，可以根据需要用 `-p` 指令映射成宿主机的端口。

基本上就是这么简单，如果你想指定更多的参数，我这里并没有提供相应的支持，但是你可以通过 fork 并修改我的项目来快速的得到你想要的结果：

[https://github.com/andyzhshg/trojan-privoxy](https://github.com/andyzhshg/trojan-privoxy)

docker hub 上的地址为：

[https://hub.docker.com/r/andyzhshg/trojan-privoxy](https://hub.docker.com/r/andyzhshg/trojan-privoxy)



