title: 让Windows开机自动运行frpc
date: 2019-04-05 00:10:00
tags: [frp, Windows, 技术]
categories: 泛技术

------

公司有一台Windows的机器，时不时需要在外边连上去做些事情，但是公司的网络是无法对外开发端口的，所以没有办法直接用远程桌面连接。一种办法是用 TeamViwer 来远程连接，但是 TeamViewer 需要要求公司这台电脑始终打开着 Teamviewer，另一方面，TeamViewer 本身的使用体验还是比直接用远程桌面差了很多。恰好我自己家的NAS上装了一个frp服务，所以我想何不在 Windows 上装一个 frpc 的客户端，把远程桌面的端口转发出去呢。

frp服务和frpc客户端的安装和配置不在本文的

