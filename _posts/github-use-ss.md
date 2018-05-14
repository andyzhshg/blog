title: 如何让git命令通过ss拉取github代码
date: 2018-05-14 22:26:00
tags: [技术, git, ss]
categories: 技术

------

`github` 还没有被墙，但是 `github` 很慢（至少我家的北京电信宽带下情况是这样）。终于忍受不了`github` 10~50 KB 的下载速度，我在网上找了下设置代理的方法，让 `git` 命令可以通过 `ss` 代理拉取 `github` 的代码。

<!--more-->

我们从 `clone` 一个全新的项目开始。

> 这里其实是针对特定项目设置代理的方法，其实也可以设置全局的代理，这样每个项目就跟不设置代理时一样操作就可以了，我不想每个项目都走代理（因为我有一些不托管在 `github` 的项目），所以才会分开设置。设置全局代理的方法，可以参考我在文末给出的参考链接。

## 1. 构建项目目录

一般情况下，我们克隆一个项目都是直接通过 `git clone` 命令，像这样：

```bash
git clone git@github.com:YourName/YourRepo.git
```

或者这样

```bash
git clone https://github.com/YourName/YourRepo.git
```

`git clone`命令会创建一个目录并将项目的代码数据拉取到这个目录中，这样的话我们还没有机会给项目设置代理，就已经开始从网络获取数据了。所以这里我将这个步骤做了一下分解。

我们通过下边的步骤来初始化项目

```bash
# 首先创建一个空的项目目录
mkdir YourRepo
cd YourRepo
# 初始化git环境
git init
# 添加远端分支
git remote add master git@github.com:YourName/YourRepo.git
# 或者
git remote add master https://github.com/YourName/YourRepo.git
```

这样我们就初始化好了一个项目，下一步我们将为这个项目设置 `ss` 代理。

## 2. 设置 `ss` 代理

设置代理的方式很简单就一条命令：

```bash
git config http.proxy 'socks5://127.0.0.1:1080'
```

上面对应的是通过http协议的方法，对应git协议，通过下边的命令：

```bash
git config core.gitProxy  'socks5://127.0.0.1:1080'
```

## 3. 拉取代码

```bash
git pull
```

到这里基本上就讲完了，如果是一个已经存在的项目，之前没有走代理，现在想走代理，那么其实更简单，略过第1步，从第2步开始即可。

> 如果是新建项目，执行完 `git pull` 后可能发现目录是空的，这是因为项目此时没有在任何一个指定的分支下，只要执行形如 `git checkout master` 命令来把项目切到一个分支即可。



[参考]: http://www.afox.cc/archives/404	"Git搭配shadowsocks使用代理访问github"

