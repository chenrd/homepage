---
categories: java
date: 2017-09-23
description: 'java基于线程安全提供了一系列的安全类，目录java.util.concurrent.*'
keywords: java, lock, 锁，concurrent
layout: post
status: public
title: 线程安全-[concurrent包]
---

目录：

- [Executor执行器](#executor)


### Executor


执行器有两个分支：ExecutorService(普通线程池执行器)，ScheduledExecutorService(调度执行器, 继承普通线程执行器)

- ExecutorService, 实现类为：ThreadPoolExecutor

- ScheduledExecutorService, 实现类为：ScheduledThreadPoolExecutor，继承了ThreadPoolExecutor

> 重点：java为我们提供了快速创建线程或调度器的方法，java.util.concurrent.Executors里面提供了一系列的方法

下面是Executor，线程池与调度器的模型图：

![concurrent-01](http://chenrd.me/images/posts/concurrent-01.png)

#### ThreadPoolExecutor

