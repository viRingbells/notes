---
title: 阅读
date: 2018-05-24 20:04:26
categories:
  - 理论
tags:
  - 学习
---

## 日志

> 原文：[The Log: What every software engineer should know about real-time data's unifying abstraction](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)
> 中文总结：[学习笔记：The Log（我所读过的最好的一篇分布式技术文章）](http://www.cnblogs.com/foreach-break/p/notes_about_distributed_system_and_The_log.html)

最近在研究MQ，在调研Kafka的时候发现了这个文章，作者是Kafka的主要作者Jay Kreps。这篇文章篇幅有点大（英文苦手可以参考一下中文总结），主要阐述了日志的本质，以及一些应用的理念。我还没有读完，不过从已经读了的部分就可以断定是必读内容。

## 《Redis设计与实现》

> 京东：[《Redis设计与实现》](https://item.jd.com/11486101.html)

最近在看《Redis设计与实现》一书，该书从全面而细致的讲解了Redis数据库的内部实现。是作为学习和了解Redis特性不可多得的好书。更重要的是，从这本书中，可以更深刻的体会到Redis的作者在实现高性能存储服务中的思考方式。学习这种思维方式，能有益于自身的技术提升。

## LVS介绍

> 简书：[LVS原理介绍](https://www.jianshu.com/p/8a61de3f8be9)

LVS，全称Linux Virtual Server，一个基于四层、具有强大性能的反向代理服务器。

## 四层、七层负载均衡的区别

> 简书：[四层、七层负载均衡的区别](https://www.jianshu.com/p/fa937b8e6712)

接上面，什么是四层反向代理。通过这篇文章可以基本了解一下。概括来说，四层就是基于TCP层（OSI第四层）的，七层就是基于应用层（OSI第七层）的。
