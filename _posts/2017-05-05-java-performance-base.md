---
categories: Java性能篇
date: 2017-05-05 16:56
description: 'java 基础性能解析，集合优化选择等'
keywords: java,性能基础,字面量,集合选择及优化
layout: post
status: public
title: Java性能基础
---

## java原理
-静态方法代替实例方法（实例方法需要维护一张类似虚拟函数导向表）
-局部变量的引用是在栈里面创建，实例变量或者成员变量，一个是在堆里面，一个实在方法区（静态区）里创建，调用速度，来说局部变量要远远高与后两个
测试：写一个最简单的调用测试用例：调用次数到亿级别之后，静态变量跟实例变量的访问速度几乎相等，局部变量，当调用次数非常大时，访问速度远优于前面两个（10亿的调用次数，差距在50毫秒级别，这个速度跟硬件有关）
-位运算的效率远远高于算数运算
-二维数组的访问速度要优于一维数组，但是二维数组比一维数组占用更多的内存空间，大概是10倍左右
-数值字面量，java默认的字面量是十进制
    十六进制：0x开头后面跟值
    八进制：0开头
    二进制：0b开头
-Java的IO尽量使用缓冲来进行（buffer），默认情况下，inputStream,OutputStream每次只操作一个字节

## 集合选择及优化
-ArrayList：数组集合，通过数组坐标索引，可以快速的找到元素，迭代速度快，由于是数组实现的，数组的长度是不可变的，所以当内存数组存满之后必须创建一个更多的数组再拷贝原来的数组到新的数组，当集合非常大的时候，添加元素效率不高
-LinkedList：链表集合，添加效率好，获取元素必须重链表头遍历，所以效率很差，遍历LinkedList必须用自身的迭代器，get(Index)方法等于又去遍历一遍。
-Vector:与ArrayList相同的实现，不同在于，Vector是线程安全的，它的所有方法都加了线程同步：synchronized
-HashTable:与HashMap的区别在于HashTable的键跟值都必须是非空的。同时HashTable也是线程安全的
-HashMap、HashTable:实现方式，类似与数组的实现，通过Map的Key.hashCode()方法得到一个坐标索引，指定到具体的值，与数组不同的地方，数组一个坐标指引只有一个值，Map的key在发生hashCode冲突的情况下会在同一个坐标下有多个值，key所指定的位置就不是一个Value，是一个桶。这个桶里面放了同一个hashCode的Entry实例，map.get方法先锁定到这个桶在遍历桶的所有元素通过entry.key找到指定的实例
-Map补充说明：影响map性能的关键因素，初始化容量，加载因子（容量达到多满之后增加容量rehash）
-LinkedHashMap：在HashMap的基础上，外面再加一层链表结构，迭代，默认有序进行
-TreeMap：实现SortedMap接口，提供排序的api，TreeMap的Key重新Object.compareTo方法，调用TreeMap方法进行排序

## 字符串优化
-java虚拟机中有一个String池，当程序直接用a = ""这种形式创建String的时候，程序会先在String池找是否存在这个字符串，没有的话会创建一个，java堆中保存的就是一个地址，指向这个String池中字符串所在的位置,用new关键字不会到String池找，直接在堆中创建，intern()方法回去String池创建
-用StringBuffer,StringBuilder的append方法可以减少创建临时的字符串
-subString:jdk1.7以前，subString会服用原来的chart数组，String内存内部用偏移量及长度来识别这个新的字符串
-split:大文本的切割建议使用StringToKenizer性能差距很大

## Java引用类型
-java的四种引用类型：强引用，弱引用（SoftReference）、弱引用（WeakReference）、虚引用（PhantomReference）
