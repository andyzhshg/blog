title: 管理多个github账号的 ssh key
date: 2018-05-05 16:14:00
tags: [github, git, ssh, 技术]
categories: 技术

------

我们大多时候是通过`ssh key`的方式来进行`github`代码库的权限管理，如何生成一个`ssh key`以及如何在`github`设置网络上有各类的说明，不是本文的重点。本文要解决的是在一个机器上管理多个账号的方法。

出于各种原因，有些人会有多个github账号，比如一个个人账号，一个工作账号。而github是不允许两个账号出现相同的SSH KEY的。那么问题来了，我们为了方便，往往都是用`ss-keygen`命令，一路默认参数在`~/.ssh`目录下生成一对名为`id_rsa`和`id_rsa.pub`的密钥，然后把`id_rsa.pub`贴到github的`SSH and GPG keys`设置中去。

如何生成一个新的密钥给另一个账号，并且在使用的过程中尽量减少麻烦呢，我这里给出一种相对简便的方法。

<!--more-->

## 1. 生成一对命名的`ssh key`

首先生成一对新的`ssh key`，依然是用`ssh-keygen`命令，只是这次不用默认的参数。

```bash
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/YourHomeDir/.ssh/id_rsa):/YourHomeDir/.ssh/account1
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /YourHomeDir/.ssh/account1.
Your public key has been saved in /YourHomeDir/.ssh/account1.pub.
The key fingerprint is:
SHA256:...
The key's randomart image is:
+---[RSA 2048]----+
| . .o+.oo        |
|. oo. ..ooo o    |
|o+o.+o .oo.+ =   |
|=+ .o.Eo... = .  |
|+    .  S. o .   |
| .    o . o      |
|       B *       |
|      o X *      |
|      .o B..     |
+----[SHA256]-----+
```

需要关注的是上边命令中的第3行，我们输入了`/YourHomeDir/.ssh/account1`，也就是我们所希望的ssh密钥的名字以及路径，其他步骤基本都一样，一路默认参数回车就可以了。这是我们在`/YourHomeDir/.ssh/`路径下生成了一对名为`account1`和`account1.pub`的新秘钥。

> 一定要注意新秘钥的命名，不要覆盖掉旧的秘钥造成不必要的麻烦。



## 2. 更改本地的SSH配置

```bash
cd ~/.ssh
touch config
vim config
```

上面的命令在ssh配置目录创建(如果不存在)一个`config`文件，并用`vim`打开编辑。通过`vim`编辑加入如下配置：

```ini
# 配置示例1
Host xxxx
    HostName github.com
    IdentityFile ~/.ssh/account1
```

其中第1行中的`xxxx`是一个代替github.com的名字，你可以用一个自己比较容易记得域名，比如我就比较喜欢这样：

```ini
# 配置示例2
Host my-github-name.github.com
    HostName github.com
    IdentityFile ~/.ssh/account1
```

其中`my-github-name`是对应我生成的这个`ssh key`的`github`账号的名字。



## 3. 将新生成的`ssh key`加到github账号配置下

将第一步生成的秘钥对中的`account1.pub`的内加入github账号的`SSH and GPG keys`设置项中。因为是一个全新的秘钥，自是不会再出现添加不进去的问题。



## 4. 克隆新的项目

一般情况下，我们是通过如下的方式克隆一个项目：

```bash
git clone git@github.com:your-account/your-prj.git
```

我们需要对这个语句中的域名部分做一下修改：

```bash
# 对应配置示例1
git clone git@xxxx:your-account/your-prj.git
```

```bash
# 对应配置示例2
git clone git@my-github-name.github.com:your-account/your-prj.git
```

这时，我们就是通过新的`ssh key`来`clone`的代码，在此之后的操作就没有区别了，一切按照之前的使用习惯即可，无论是`pull`还是`push`代码等操作都使用新的`ssh key`来进行了。

> 这里补充说一个可能跟`ssh key`的关系不大，但是跟多账号有关的问题，是关于`commit`代码的账号的设置的。如果默认不处理，提交代码的时候提交信息中的用户和邮箱信息是用户设置的全局账户的信息，当时应该是这样设置的：
>
> ```bash
> git config --global user.name "You Name"
> git config --global user.email name@example.com
> ```
>
> 我们往往是要给不同项目设置不同的提交信息，毕竟你不想把公司的邮箱带到私人项目的提交记录中去。可以通过下边的方式文每个项目单独设置提交账户信息：
>
> ```bash
> cd YourRepoPath
> git config user.name "You Name"
> git config user.email name@example.com
> ```
>
> 其实很简单，就是去掉`--global`参数。

------

我的第一个账号是通过默认方式添加的，所以如果没有用自定义域名添加的项目都是使用的默认的密钥即`id_rsa`，为了使用方便，可以让自己使用最频繁(或者是项目最多的账户)使用这个默认配置。

如果你有更多的账号，通过上边的方法来生成更多的`ssh key`并通过自定义域名的方式对不同账号的项目进行区分即可。

本方法同样适用于`gitlab`的多账号情境。