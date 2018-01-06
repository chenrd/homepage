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

构建线程池有几个重要的属性：

```

private volatile ThreadFactory threadFactory;

//请求线程池执行任务被拒绝之后的处理策略，
private volatile RejectedExecutionHandler handler;

//线程池核心大小，保持线程池线程的数量，及时线程是空闲的也会保存在线程池中
private volatile int corePoolSize;

//最大线程数量，默认Integer.MAX_VALUE, 配合keepAliveTime及unit属性，线程数量大于核心数量时，在空闲一段时间之后会关闭线程(keepAliveTime)
private volatile int maximumPoolSize;

//当所有的线程都处于工作状态时，指定线程的
private final BlockingQueue<Runnable> workQueue;
private volatile long keepAliveTime;

//线程池构造方法
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                              BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```

> 重点：讲一下RejectedExecutionHandler，java为线程池Rejected提供了几种策略
> 重点：默认的是ThreadPoolExecutor.AbortPolicy，抛出异常

- ThreadPoolExecutor.CallerRunsPolicy 使用当前线程运行（执行：executor.execute(Runnable)的线程），完美的循环，永远不会有空闲的线程，线程池不够就用当前线程执行代码
- ThreadPoolExecutor.AbortPolicy 抛出拒绝任务的异常，throw new RejectedExecutionException, 默认
- ThreadPoolExecutor.DiscardPolicy 不做处理，直接丢弃任务（Runnable）
- ThreadPoolExecutor.DiscardOldestPolicy 丢弃等待队列中的一个任务（Runnable），把当前的Runnable加入队列，（注意初始化线程池的队列形式）
