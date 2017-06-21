---
categories: redis
date: 2017-06-21 18:21
description: 'redis执行效率测试'
keywords: redis,缓存
layout: post
status: public
title: redis性能测试
---

## 前言  
>现在有一个需求要用redis实现分布式锁，查过资料之后了解到实现方式：用setnx,getset特性来实现  
>>setnx(look.key, System.currentTimeMillis())，假设线程T1,通过setnx方法获取到锁，T2,T3返回0没有获取到，这个时候T2,T3同时用get方法获取
currentTimeMillis(time1),如果time1没有过期，T2,T3继续循环，至到look.key被del,获取过期。如果已经过期，那么T2,T3同时执行getset(look.key, System.currentTimeMillis())，
返回一个time2。如果T2的time1=time2说明T2比T3先得到锁，T3继续循环。按上面的逻辑更多线程的情况下类似。  

>>按照上面的方法有一个问题，因为不是阻塞式的锁，所以需要while循环中 sleep(millis)不断的去重试获取锁，那么这个millis的值设置多少合适呢  

>>测试多线程执行10W数据的插入需要多长时间
    public class RedisTest {
        public static int number = 0;
	
        public static void main(String[] args) {
            System.out.println(System.currentTimeMillis());
            for (int i = 0; i < 100; i++) {
              ThreadTest test = new ThreadTest(i);
              test.start();
            }
        }
    }

    class ThreadTest extends Thread {
	
        private int index;

        public ThreadTest(int index) {
            this.index = index;
        }

        public void run() {
            Jedis jedis = new Jedis("192.168.2.99", 6330);
            long currentTime = System.currentTimeMillis();
            for (int i = index * 1000; i < 1000 *(index + 1); i++) {
                jedis.set("test_" + i, i + "");
            }
            System.out.println(Thread.currentThread().getName() + ":" + (System.currentTimeMillis() - currentTime));
            jedis.close();
            synchronized (RedisTest.class) {
                RedisTest.number++;
                if (RedisTest.number == 100) {
                    System.out.println(System.currentTimeMillis());
                }
            }
        }
    }
上面代码的执行结果如下：  
Thread-97:2825  
Thread-61:2826  
Thread-5:2827  
Thread-89:2827  
Thread-81:2827  
Thread-57:2829  
Thread-25:2833  
1498040064572  
结论：单个线程执行1000条数据的插入大概要2800毫秒的时间，所有线程执行完成大概也是2800毫秒，补充一下redis服务器部署在内网，测试运行在本机，通过内网链接
，可以看得出来基本上redis的插入效率可以说是在2800毫秒内完成了10W数据的插入，但是Java程序执行单个线程执行1000条数据的插入就需要2800毫秒，

>>测试单个线程执行10W数据的插入  

    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.2.99", 6330);
        long currentTime = System.currentTimeMillis();
        for (int i = 0; i < 1000 ; i++) {
            jedis.set("test_" + i, i + "");
        }
        System.out.println(Thread.currentThread().getName() + ":" + (System.currentTimeMillis() - currentTime));
        jedis.close();
    }
    
执行结果：main:1323  
从上面单线程及多线程的情况基本可以得出的结论，程序单线程执行能力是插入一条数据需要1.3毫秒，redis缓存10W数据需要2800(35717.81)毫秒（这个基本差不多，官网上面说redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q，测试可以达到10W，其实我的测试服务器只有56818.18）
3571.78与56818.18之间的差值就是查询运行时的时间损耗，当然这个损耗肯定是线程数越多，那么损耗就越低
（测试了一下1000个线程同时启动，完成10W条数据的插入用时：2188，但是单个线程的完成时间增加（同时竞争redis的线程））。  

从上面的结果我们可以来总结一下sleep(millis)怎么设置，并发线程数 * (1000 / (millis + 2.8))  < 35717.81，这个公式里面的2.8是近似的值，不同线程数也不同  
1000个线程开启的时候最高达到4，所以这个2.8应该是取值在1.3以上  

上面得出的结论还有依赖与程序运行的服务器处理能力，得出的2.8与35717.81是不同的。  
