title: 实现一个交易所撮合引擎
date: 2019-05-13 14:16:00
tags: [blockchain, go, 区块链, 技术,算法,架构]
categories: 技术

------

最近在做一个交易所撮合引擎的项目，中间从设计到实现确实遇到了不少的问题，踩了很多坑，也积累了很多的经验。所以这里想写一系列文章来总结一下。

因为是工作的项目，项目的细节和代码都不能公开，所以我打算把这个设计和实现的过程重新走一遍，原项目是用 `golang` 来实现的，这里我打算用 `C++` 来重新实现。

<!-- more -->

## 用户数据库

余额 活动/锁定

挂单

成交记录

## 下单

~~id 生成~~ / 定序

数据持久化

传递到撮合引擎



并发 / 多活 / 一致性



## 撮合

主备方案



## 清算





## 行情