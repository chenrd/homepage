---
categories: java
date: 2017-09-23
description: 'java cas原子性,concurrent包的安全性解析'
keywords: java,java 1.8,设计模式
layout: post
status: public
title: java cas,concurrent包的安全性解析
---

开始之前来了解一下锁的几个概念
- 公平锁/非公平锁
- 可重入锁
- 独享锁/共享锁
- 互斥锁/读写锁
- 乐观锁/悲观锁
- 分段锁
- 偏向锁/轻量级锁/重量级锁
- 自旋锁

#### 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁。

非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。

Synchronized是一种非公平锁。

ReentrantLock通过构造函数指定该锁是否是公平锁。

#### 可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。

- ReentrantLock是可重入锁
- Synchronized是可重入锁
```
synchronized void setA() throws Exception{
    Thread.sleep(1000);
    setB();
}

synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
```

#### 独享锁/共享锁

独享锁是指该锁一次只能被一个线程所持有。

共享锁是指该锁可被多个线程所持有。

- 写锁肯定是独享锁

- 读锁可以是共享锁

- Synchronized不分读写，所以是独享锁

#### 互斥锁/读写锁

上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。

互斥锁在Java中的具体实现就是ReentrantLock

读写锁在Java中的具体实现就是ReadWriteLock

#### 分段锁

分段锁其实是一种锁的设计，并不是一种具体的锁，ConcurrentHashMap就是通过分段锁的设计实现高效的并发操作（类似与数据库表格锁与行锁，一种让并发更具性能的思路）

java.util.concurrent是java1.5之后引入的包，主要是已乐观锁的概念

### 实践:ReentrantLock

下面是从dubbo com.alibaba.dubbo.remoting.exchange.support.DefaultFuture拷贝出来的一段代码

```
public Object get(int timeout) throws RemotingException {
    if (timeout <= 0) {
        timeout = Constants.DEFAULT_TIMEOUT;
    }
    if (! isDone()) { // 判断异步是否已经返回
        long start = System.currentTimeMillis();
        lock.lock(); //锁定，同一个DefaultFuture实例，调用get的方法会阻塞
        try {
            while (! isDone()) { //while循环里面添加线程等待
                done.await(timeout, TimeUnit.MILLISECONDS); //等待时间
                if (isDone() || System.currentTimeMillis() - start > timeout) {
                    break;
                }
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
        if (! isDone()) {
            throw new TimeoutException(sent > 0, channel, getTimeoutMessage(false));
        }
    }
    return returnFromResponse();
}
```

