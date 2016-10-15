title: 巧用Xcode的Target管理开发和生产的APP版本
date: 2015-09-07 19:06:01
tags: [iOS, 技术]
categories: 移动开发

---

APP开发过程中我们经常会碰到这样的情况：需要同时维护两个版本，一个是开发测试版本，一个是线上运行版本。两者需要有不同的Bundle ID，有时甚至要连接不同的服务器。

遇到这种情况，我们的做法往往是平时开发用测试版本的Bundle ID，测试的服务地址；上线的时候人工修改成正式版本的Bundle ID，线上的服务地址。

其实，Xcode的Target功能能够很好地解决这个问题。

<!--more-->

我们新建一个项目来说明怎么做，项目的名字叫`MultiTargetPrj`。

Xcode默认给我们创建了两个Target，一个是`MultiTargetPrj`，另一个是`MultiTargetPrjTests`。`MultiTargetPrjTests`是单元测试的Target，我们暂且忽略。

我们用`MultiTargetPrj`作为正式版，在此基础上我们新建一个Target来做开发版应用。如下图，在已有的`MultiTargetPrj` Target上点击右键，在菜单中选择`Duplicate`。

![复制targer0](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/duplicate0.png)

这样Xcode就为我们创建了一个名为`MultiTargetPrj copy`的Target，如下图

![复制targer1](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/duplicate1.png)

后边带个`copy`尾巴的名字太不优雅了，让我们重命名一下，要重命名的地方不少，我们还是直接看图吧：

![重命名0](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/rename0.png)

然后重命名Schemes：

![重命名1](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/rename1.png)

![重命名2](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/rename2.png)

最后然我们编辑一下info.plist，让应用在屏幕上显示的名字各不相同，正式版叫MT，开发版叫MT Dev，如下图所示：

![显示名0](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/display0.png)

![显示名1](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/display0.png)

我们在界面上放置一个label，预期在运行正式版的应用时，显示`正式版`，测试版的应用时，显示`开发版`。

![xib](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/xib.png)

为了让程序可以区分正式版和测试版，我们给开发版的target中设置一个预定义宏`MULTITARGET_DEV`:

![设置预定义宏](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/macro.png)

然后我们用一段代码来区分版本，设置不同的版本的文字：

``` objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
#ifndef MULTITARGET_DEV
    self.label.text = @"正式版";
#else
    self.label.text = @"开发版";
#endif
}
```

分别运行一次两个Target，我们看到的主屏是这样的：

![主屏](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/home.png)

两个Target的运行结果是这个样子的：

![正式版截图](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/run0.png)

![测试版截图](http://up4dev.oss-cn-qingdao.aliyuncs.com/multitarget/run1.png)

我把这个测试程序的代码放到了GitHub: [andyzhshg/MultiTarget](https://github.com/andyzhshg/MultiTarget)

