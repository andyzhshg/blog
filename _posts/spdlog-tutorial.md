title: spdlog 简介
date: 2018-07-27 18:09:00
tags: [C++,技术,算法,库,后端]
categories: C++

---

spdlog是一个速度快，只有头文件(header only)的C++日志库。安装和使用非常简单，而功能也比较强大，因为其简单易用轻量，可以用作我们日常开发的日志库。

<!-- more -->

项目的地址：[https://github.com/gabime/spdlog](https://github.com/gabime/spdlog)

文档地址：[https://github.com/gabime/spdlog/wiki](https://github.com/gabime/spdlog/wiki)

项目主页上介绍了该库的一系列特色：

- 快，非常快，性能是设计的首要目标。
- 只有头文件 (header only)，拷贝即用
- 基于[fmt](https://github.com/fmtlib/fmt)实现的丰富的格式调用
- 自定义格式
- 条件日志输出
- 多线程/单线程日志输出
- 丰富的日志输出目标
  - 日志文件自动切割(Rotating)
  - 按日分割日志文件
  - 控制台输出(支持着色)
  - 系统日志(syslog)
  - windows debuger (`OutputDebugString(..)`)
  - 方便的自定义扩展
- 过滤-可以再运行时或者编译时修改过滤级别



使用的例子可以参考项目的实例或者文档，非常明了。





