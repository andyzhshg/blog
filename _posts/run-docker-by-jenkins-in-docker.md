title: 用安装在 Docker 中的 jenkins 运行 Docker 任务
date: 2018-11-27 11:57:00
tags: [技术,docker,ci,jenkins,]
categories: 技术

------

题目有点绕，我先尝试翻译成人话——首先安装一个 jenkins，这个 jenkins 是通过 docker 安装的，然后要用这个 jenkins 来运行自动化的项目，项目中会用到 docker 命令。而这种工作场景，采用默认的配置，是完成不了的。

<!--more-->

## 缘起

故事的缘起是这个博客，他是基于 Github Pages 的，使用的过程就是用 MarkDown 写文章，通过 HEXO 系统来构建，然后 push 到 github 上。

整体的流程实际上也不算复杂，但是作为一个程序员，总觉得会有更加智能和省事的方法（实际上就是不折腾不舒服而已）。

所以我想构建一个系统，来达到博客的自动构建和发布的效果。鉴于这部分跟主题的关系不是特别大，还是不再赘述博客的事了，也许以后会写一篇博客自动构建的文章，这里就不展开了。

## 思路

提到自动构建，很自然的想到了jenkins；然后恰好我有一台群晖的NAS，上边可以跑Docker，所以又很自然的想到了通过Docker来安装jenkins；有了jenkins，当然还要通过jenkins来运行自动化的任务，运行任务最简单且对NAS系统侵入最小的方式是通过Docker运行任务。

## 问题

上面的思路看似很顺畅，但是认真思考一下会发现，因为jenkins本身是运行在一个容器里的，所以我们在创建任务的时候给出的脚本或命令，都是在这个容器内运行的，这就出现了一个问题，这个容器内部是没有docker环境的，所以执行不了docker命令，也就是说，必须想办法来让这个容器内部可以执行docker任务。

## 方案

经过一番搜索之后，我找到了了这篇文章：[在（Docker里的）Jenkins里运行Docker](http://www.dockone.io/article/431)。

最终我选择了文中提到的`DooD（Docker-outside-of-Docker）`方案。

也就是所想办法让jenkins容器可以执行宿主机上的docker命令。

因为需要给予jenkins用户sudo权限，然而官方的镜像jenkins默认是没有sudo用户权限的，所以我在官方镜像的基础上新建了一个镜像，默认给jenkins用户sudo权限。

[andyzhshg/jenkins](https://hub.docker.com/r/andyzhshg/jenkins/)

改动的内容如下：

```dockerfile
FROM jenkins:2.60.3
MAINTAINER andyzhshg <andyzhshg@gmail.com>

USER root
RUN apt-get update \
  && apt-get install -y sudo \
  && rm -rf /var/lib/apt/lists/*
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers

USER jenkins
```

运行这个镜像的容器很简单：

```bash
docker run -d \
	-v /var/run/docker.sock:/var/run/docker.sock \
	-v $(which docker):/usr/bin/docker 
	-p 8080:8080 \
	andyzhshg/jenkins
```

需要重点关注带有 `-v` 参数的两行：

第1行是将宿主机的 `/var/run/docker.sock` 映射到容器中，这样在容器中运行的 `docker` 命令，就会在宿主机上来执行。

第2行是将宿主机的 `docker` 程序映射进容器中，这样本身没有安装 `docker` 的 `jenkins` 容器就可以执行 `docker` 命令了（事实上容器里是没有运行 `docker` 的服务的，我们只是用这个映射进容器的 `docker` 来作为客户端发送docker的指令到 `/var/run/docker.sock` 而已，而 `/var/run/docker.sock` 已经被链接到宿主机了）。

至此，我们的 `jenkins` 就准备就绪了。

> 当然，通常情况下我们还需要把jenkins自身的数据目录链接到宿主机的目录中，以保证容器被销毁后还能够再启动新的数据而数据可以得到保留，这些可以参考jenkins官方镜像的说明。
>
> 比如在`docker run`加上 `-v your_jenkins_data_path:/var/jenkins_home`参数。

初始化jenkins的配置之后，我们就可以在jenkins上来新建一个自动化的构建项目了，比如我的blog自动构建项目的自动化脚本大概就是这个样子：

```bash
echo "new blogs posted, begin auto build."

# 从gitlab拉取内容更新，构建并推送到github pages
sudo docker run --rm -v /volume1/docker/blog/key/:/root/.ssh -v /volume1/docker/blog/up4dev:/blog andyzhshg/hexo

# 推送一份内容原文到github做备份
sudo docker run --rm -v /volume1/docker/blog/key/:/root/.ssh -v /volume1/docker/blog/up4dev:/blog andyzhshg/hexo /blog/backup2github.sh 


echo "auto build done."
```

这虽然是一个特例化的应用场景，但是其他的项目也大同小异，只要准备好docker的镜像，然后再jenkins执行`docker run`就可以了。

需要注意的是，这里的`docker run`一定是要以sudo权限运行的。

