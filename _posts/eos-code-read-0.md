title: EOS 代码分析 [0]
date: 2018-06-26 15:32:00
tags: [区块链, EOS, 智能合约, 技术, C++]
categories: 技术

------

去年就开了一个坑打算分析比特币的源码，结果到现在还没有动手，理由能找到不少，但归根结底还是因为懒。比特币源码分析这个坑不打算弃，因为毕竟代码已经读的差不多了，就差写文章了，如果我争气一点的话会找时间把这个坑填上。

最近都在研究EOS，又手痒打算写一系列EOS源码分析的文章，这算是个开端。为了避免出现去年那样的挖坑不填的情况，我打算这次边看代码边写文章，文章可能会很杂很细碎，只要起到一个记录的作用就好了。如果以后有余力，再写一写系统性的分析文章。

FLAG立在这里了，开始执行。

以下是已经完成的文章列表，我随着写作的进度逐步添加：

- [EOS 代码分析 [1] —— AppBase](http://www.up4dev.com/2018/06/26/eos-code-read-1-appbase/)

<!-- more -->

我从eos的主项目clone了一份代码，写这篇文章的时候的最新release版本是v1.0.6，后续如果有重要的更改可能会merge主项目的代码，本系列文章讨论的代码均会指向我的这个fork的项目：

**[andyzhshg/eos](https://github.com/andyzhshg/eos/tree/v1.0.6) **

这篇开头的文章先简单介绍一下代码的基本结构，我大致介绍一下代码根目录的下重要的文件和目录：

| 文件或目录                                                   | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [programs](https://github.com/andyzhshg/eos/tree/v1.0.6/programs) | 重要的二进制程序的入口代码，每个子目录都对应一个二进制程序，比如服务主程序 [nodeos](https://github.com/andyzhshg/eos/tree/v1.0.6/programs/nodeos)，钱包程序 [cleos](https://github.com/andyzhshg/eos/tree/v1.0.6/programs/cleos) 等 |
| [libraries](https://github.com/andyzhshg/eos/tree/v1.0.6/libraries) | 重要组件库代码，可以说核心的实现代码大都在这个路径下         |
| [contracts](https://github.com/andyzhshg/eos/tree/v1.0.6/contracts) | 合约程序代码，包含重要的系统合约和一些示例合约               |
| [plugins](https://github.com/andyzhshg/eos/tree/v1.0.6/plugins) | 插件目录，eos 的架构本身就是基于一套插件体系构建起来，功能都是以插件的形式集成进来的，所以这个目录非常重要 |
| [eosio_build.sh](https://github.com/andyzhshg/eos/blob/v1.0.6/eosio_build.sh) | 构建脚本，通过该脚本可以自动获取依赖构建项目                 |

