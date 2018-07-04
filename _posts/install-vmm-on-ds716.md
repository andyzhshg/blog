title: 在群晖 DS716+II 上安装 VMM(Virtual Machine Manager)
date: 2018-05-10 13:12:00
tags: [NAS, 技术, 虚拟机,VMM]
categories: 泛技术

------

我有一台群晖的NAS，型号是DS 716+II。一直都知道群晖的高端型号都是支持虚拟化的，但是在群晖的官网上查Virtual Machine Manager的支持机型，我的716+II是不支持的。不过因为这个机器是支持Docker的，所以不支持虚拟机就不支持吧，反正大部分我想要的功能用Docker都实现，这机器的赛扬处理器用来跑虚拟机本身也不会跑的很舒服。

不过最近在研究如何自动化的把DS Photo上的照片再备份一份到Google Photos，毕竟Google Photos的体验还是很不错的。

网上查到有人说了一种曲线救国的方案：用VMM虚拟一个Windows，然后在Windows里安装Google的Windows版的客户端。虽然蛋疼，但也是一种曲线救国的方案，于是我开始研究怎么在我的机器上安装VMM。然后发现居然非常简单，直接下载一个916+(其他x86平台的安装包应该也都可以)的VMM的安装包，手动安装即可...

我做一下雷锋，直接把916+的VMM安装包的下载页面贴这里，免去大家自己找的麻烦：

[https://www.synology.cn/zh-cn/support/download/DS916+#packages](https://www.synology.cn/zh-cn/support/download/DS916+#packages)

在这个页面里搜索`Virtual Machine Manager`即可找到下载链接。至于怎么手动安装，怎么使用VMM，我就不详细说了。





