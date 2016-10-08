title: git修改提交记录的用户名和邮箱
date: 2016-10-08 19:00:00
tags: [技术, git]
categories: 技术

---

有时候，我们进行了一次commit之后发现，用户名和邮箱错了。为什么会有这种情况，往往我们在公司和个人的项目中，会使用不同的名称和邮箱，这样一来，电脑中就有了两套用户名和邮箱的配置，或者是公司的是默认，或者是个人的是默认的，但是开始一个新项目的时候，如果正好忘了修改项目的配置，就会出现提交用户不正确的情况。

一旦出现了这种情况的话，该怎么处理呢？

如果这次提交是你的最后一次提交，那么很简单，通过下面的命令修改最后一次提交的用户名和邮箱地址：

```bash
git commit --amend --author='yourname <yourname@email.com>'
```

如果已经推送到了远端服务器，通过下面的命令将修改强制推送到远端服务器就可以了：

```bash
git push origin develop -f
```

----------------

*参考文献* 

- [StackOverflow: Change commit author at one specific commit](http://stackoverflow.com/questions/3042437/change-commit-author-at-one-specific-commit)
- [Git 基础 - 撤消操作](https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%92%A4%E6%B6%88%E6%93%8D%E4%BD%9C)


