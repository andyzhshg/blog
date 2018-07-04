title: EOS 代码分析 [1] —— AppBase
date: 2018-06-26 15:35:00
tags: [区块链, EOS, 智能合约, 技术, C++, 插件]
categories: 区块链

------

[EOSIO/appbase](https://github.com/eosio/appbase),如项目介绍所说——

> The AppBase library provides a basic framework for building applications from a set of plugins. AppBase manages the plugin life-cycle and ensures that all plugins are configured, initialized, started, and shutdown in the proper order.

是一个从一系列插件构建应用的基本框架。 EOS的大多应用都是基于这个框架来构建的。

AppBase是独立于EOS项目的一个独立项目，可以单独的编译，我们也可以利用这个框架构建自己的应用。

<!-- more -->

为了方便描述，我fork了一份代码，并在这个fork项目上添加了我阅读过程中的注释，这个fork的项目在这里：

[andyzhshg/appbase](https://github.com/andyzhshg/appbase)

下文的介绍均基于AppBase提供的[示例程序](https://github.com/andyzhshg/appbase/tree/master/examples)为基础进行说明，这个示例程序实现了一个[`net_plugin`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L41)，这个[`net_plugin`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L41)又有一个依赖项[`chain_plugin`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L14)。这基本展示了AppBase使用中的方方面面。

## 0. 基本使用流程

一个基于AppBase的程序的基本使用流程如下：

1. 注册插件，[`register_plugin`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L68)
2. 初始化，[`initialize`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L70)
3. 启动，[`startup`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L73)
4. 进入事件监听等待，[`exec`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L75)
5. 程序退出，`shutdown`。

基本上就是这么简单，AppBase本身提供了一个基础进程环境，来使得用户的代码可以以插件的形式集成进来。

下面我们逐一解析一下这些步骤

## 1. 注册插件

[`application::register_plugin`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L84)是一个模板函数，模板参数是要注册的插件的类型。

该函数首先通过插件的名称查找该插件是否已经注册过，如果注册过则直接返回，不会重复注册。插件名的名称是根据插件的类型推导得来的。

如果没有注册过，则`new`一个插件的对象，并将其记录进`plugins`这个列表中。

调用插件自身的`register_dependencies`来注册插件自身的依赖，插件的依赖也是插件。

## 2. 初始化

从[`application::initialize`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L64)函数的实现可以看到实际的初始化工作是在[application::initialize_impl](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L114)完成的。

其完成的主要工作是完成程序配置项的设置和根据配置项做初始化。

配置项的初始化部分，首先是调用[插件的配置项设置](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L83) ，然后才是[程序自身的配置项设置](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L95)。配置项的处理是使用`boost::program_options`完成的。

配置项处理完成后，首先是[根据配置进行`application`自身的初始化处理](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L121)，然后是[根据配置项进行插件的初始化处理](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L195)。

## 3. 启动

[`application::startup`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L61)的过程很简单，就是逐一调用插件的[`startup`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L236)函数，并处理异常。

因为[`application::startup`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L61)本身不进行任何应用逻辑的处理，所有的应用逻辑都是插件来完成的，所以我们只要看一下插件是如何完成[`startup`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L236)的。

我们发现，用户并不需要实现`startup`函数，而是实现一个[`plugin_requires`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L241)和[`plugin_startup`](https://github.com/andyzhshg/appbase/blob/master/include/appbase/application.hpp#L244)。而事实上如果我们观察示例程序，发现其并没有实现`plugin_requires`函数，这个函数事实上是实现了的，由一个[`APPBASE_PLUGIN_REQUIRES`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L47)宏来实现，这个宏极大的简化了声明依赖的过程，只需要提供一个依赖的插件的类型的列表即可，我的代码注释中有对[这个宏的简单解释](https://github.com/andyzhshg/appbase/blob/master/include/appbase/plugin.hpp#L11)，你也可是尝试展开这个宏。经过这个宏的简化，实际上必须有用户自己完成的只有`plugin_startup`这个函数了，这里给用户一个机会来完成插件自身在启动前要完成的工作。

需要注意的是，插件的调用顺序是与注册的顺序相同的。

## 4. 进入事件等待

[`application::exec`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L229)函数启动一个`boost::asio::io_service`并监听了几个信号：`SIGINT / SIGTERM / SIGPIPE`，并阻塞在[`io_serv->run();`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L249) 这行代码，当进程收到这几个信号中的一个的时候，阻塞函数返回，并进入[`shutdown`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L210)函数。

## 5. 程序退出

观察示例代码我们发现并没有显式调用[`application::shutdown`](https://github.com/andyzhshg/appbase/blob/master/application.cpp#L210)函数，实际上这是由上一步骤中监听的事件来触发的，也就是类似`CTRL+C`这样的键盘时间或者`kill`之类的命令触发的。

在这个函数中，会逐一的调用插件的`shutdown`函数，时间上就是用户自己编写的[`plugin_shutdown`](https://github.com/andyzhshg/appbase/blob/master/examples/main.cpp#L60)函数。

需要注意的是，`shutdown`中调用插件的顺序是与`startup`的时候的顺序相反的，也就是注册的相反的顺序。

## 总结

综合上述的流程，我们发现基于`AppBase`编写一个程序主要的工作就是编写插件，而完成一个插件的流程就是如下简单几个步骤：

1. 从`appbase::plugin`派生

2. 用`APPBASE_PLUGIN_REQUIRES`宏来声明插件的依赖

3. 实现`plugin_initialize`

4. 实现`plugin_startup`

5. 实现`plugin_shutdown` 


对于AppBase的实现细节我的fork项目[andyzhshg/appbase](https://github.com/andyzhshg/appbase)里的注释解释的比较详细了。其中有一部分是关于`method`和`channel`的，`AppBase`本身的示例并没有演示用法，这篇文章没有解释，也许后续研究EOS的过程中我会回头解释这个设计。

   

