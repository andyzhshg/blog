title: 群晖 Let's Encrypt 泛域名证书自动更新
date: 2018-05-29 18:00:00
tags: [HTTPS, Let's Encrypt, NAS, 群晖,泛域名,acme,Synology,SSL]
categories: 泛技术

---

去年曾经写过一篇文章介绍如[何在群晖的 NAS 通过 acme 协议更新 Let's Encrypt 的 HTTPS 证书](http://www.up4dev.com/2017/09/11/synology-ssl-cert-update/)。最近突然发现acme协议版本更新，开始支持泛域名(wildcard)，也就是说，可以申请一个类似`*.domain.com`的单一证书，就可以适配`abc.domain.com`，`xyz.domain.com`这类的子域名，而不需要单独为每个子域名申请证书了。

[**Neilpang/acme.sh**](https://github.com/Neilpang/acme.sh) 工具很快就支持新的协议了，我这篇文章就是在这个工具的基础上，实现泛域名的自动更新。为了减少复杂度，我编写了一个一键更新的懒人脚本，来帮助不愿意了解原理的同学快速部署。

<!-- more -->

## 1. 准备工作

因为我介绍的方法是一键替换群晖的默认证书，所以，为了防止意外，最好保证你的证书列表里只有一条记录，即默认证书那一条。实际上因为支持了泛域名证书，基本上这一条记录就足够用了（当然，如果你要管理多个域名，可能本文的方法并不实用）。所以开始工作前你的证书列表大概应该是这个样子：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/cert-list.png)

## 2.  下载一键更新脚本

这是一键脚本的项目地址：[andyzhshg/syno-acme](https://github.com/andyzhshg/syno-acme)。

如果你对项目本身不感兴趣，可以直接下载打包好的工具：[syno-acme v0.1.6](https://github.com/andyzhshg/syno-acme/archive/v0.1.6.zip)。

可以通过 File Station 将下载的工具上传到NAS的任意目录下，并解压。

解压后大概是这个样子：

![解压](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/unzip.png)



## 3. 配置脚本参数

编辑脚本的配置文件`config`:

![配置](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/config.png)

如图所示，需要编辑的几个字段我用蓝框标记出来了。

首先是DOMAIN，也就是你的域名。

然后是DNS的类型，根据服务商的不同，DNS类型各不相同，比如阿里云（dns_ali），Dnspod（dns_dp），Godaddy（dns_gd）等。

最后要根据不同的服务商配置服务上提供的授权密钥等信息，比如我的域名服务商是阿里云，我需要编辑`Ali_Key`和`Ali_Secret`字段，字段的内容需要到域名服务商的管理后台来查看，因为不同的服务商的查看方式不同，请大家根据自己的实际情况去查找吧。

需要指出的是，我给出的配置文件模板并没有给出所有acme.sh支持的域名服务商，大家可以参照 [https://github.com/Neilpang/acme.sh/tree/master/dnsapi](https://github.com/Neilpang/acme.sh/tree/master/dnsapi)来添加自己的配置。一般情况下，这个页面每个文件对应一个域名服务商，比如`dns_ali.sh`就是对应阿里云，文件名去掉`.sh`扩展名就是DNS类型，比如阿里云的DNS类型就是`dns_ali`。打开对应文件， 一般都可以在文件的头部找到需要设置的授权信息对应的密钥，比如阿里云的授权密钥所在的位置如下图所示：

![APIKEY](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/apikey.png)

`config`模板中没有的服务商，请大家自行完善。

[^2018.05.31]: 针对评论区同学提出的 Linode 的 API 生效时间的问题，增加了一个配置参数`DNS_SLEEP`，出现类似问题的话可以通过修改增大这个参数来解决。

## 4. 配置定时任务

### i. 查找脚本路径

在 File Station 中定位到下载的一键脚本的目录，查看该脚本的绝对路径：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/file-info.png)

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/file-path.png)

复制完整的绝对路径到剪贴板。

### ii. 创建定时任务

打开 `控制面板 / 任务计划 / 新增 / 计划的任务 / 用户自定义的脚本`：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/task.png)

设置任务名称和操作用户，需要注意的是这里一定要选择`root`：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/task-name.png)

设置计划的时间和周期，这里只支持按月或者年重复，我们只能取按月重复才能满足 Let's Encrypt 至少3个月更新一次的要求：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/task-inv.png)

设置执行脚本，这里我们将脚本的输出重定向到了一个`log.txt`的文件中，以方便后期查看脚本的执行情况：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/task-cmd.png)

上图红框中的脚本命令为(注意没有换行)：

```bash
/volume1/nas_share/certs/syno-acme/cert-up.sh update >> /volume1/nas_share/certs/syno-acme/log.txt 2>&1
```

具体的路径是步骤 `i`中复制的路径。

### iii. 试运行脚本

可以在新建的任务上点击右键立即执行任务：

![](https://up4dev.oss-cn-qingdao.aliyuncs.com/nas-cert-up/task-run.png)

这样脚本就会运行，自动更新证书，并重启web服务器加载新的脚本。以后，NAS会每隔一个月执行一次该脚本，自动更新证书。



### iv. 回滚

脚本里提供了回滚命令，可以通过ssh登录到nas，定位到对应目录，执行如下命令回滚证书目录到备份的状态：

```bash
/volume1/nas_share/certs/syno-acme/cert-up.sh revert
```

## 总结

这个一键脚本的特点是最小限度的触碰系统文件，仅`/usr/syno/etc/certificate/_archive`目录会被更改。`acme.sh`工具随用随时下载，保持最新，用完即删除，不占用磁盘空间。

这基本就是本文的全部了，如果大家使用中遇到问题，可以在这里留言或者到 [https://github.com/andyzhshg/syno-acme/issues](https://github.com/andyzhshg/syno-acme/issues) 提issue。

------


[^参考1]: [Synology NAS Guide](https://github.com/Neilpang/acme.sh/wiki/Synology-NAS-Guide)
[^参考2]: [群晖 Let's Encrypt 证书的自动更新](http://www.up4dev.com/2017/09/11/synology-ssl-cert-update/)

