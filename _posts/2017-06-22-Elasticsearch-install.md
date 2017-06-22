---
categories: Elasticsearch
date: 2017-06-21 18:21
description: '全文检索数据库Elasticsearch开始学习'
keywords: Elasticsearch,全文检索
layout: post
status: public
title: Elasticsearch安装
---

## 说明：这里是对Elasticsearch学习的记录总结，中文文档：https://es.xiaoleilu.com/010_Intro/05_What_is_it.html  
1、安装：下载地址：http://www.elasticsearch.org/download/  
2、最新版本5.4.zip包下载过来之后，要求jdk版本要1.8以上，附上JDK各版本下载地址http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html  
3、解压之后进入目录启动：./bin/elasticsearch，后台启动./bin/elasticsearch -d  
4、配置文件在config/elasticsearch.yml  


## 学习的总结  
1、集群，节点，索引，分片，备份分片概念解析  
一、传统数据库与elasticsearch概率关系对比  
    
    Relational DB -> Databases -> Tables -> Rows -> Columns
    Elasticsearch -> Indices   -> Types  -> Documents -> Fields

Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。

二、集群的健康性green、yellow或red，通过命令GET /_cluster/health查看  
status字段提供一个综合的指标来表示集群的的服务状况。三种颜色各自的含义：  
green	所有主要分片和复制分片都可用  
yellow	所有主要分片可用，但不是所有复制分片都可用  
red	不是所有的主要分片都可用  
在接下来的章节，我们将说明什么是主要分片(primary shard)和复制分片(replica shard)，并说明这些颜色（状态）在实际环境中的意义。  

三、索引与分片的关系  
PUT /testindex,命令回去创建一个索引：testindex,创建一个索引默认会创建5个主分片，文档保存在分片当中，客户端不直接与分片交互，索引建立与分片的联系
客户端通过索引检索文档  

    PUT /blogs
    {
       "settings" : {
          "number_of_shards" : 3,
          "number_of_replicas" : 1
       }
    }  
    
创建一个索引，指定3个主分片，1个副分片（每个主分片都有1个副分片）  
