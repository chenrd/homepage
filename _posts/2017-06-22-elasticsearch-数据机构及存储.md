---
categories: Elasticsearch
date: 2017-06-22 14:54
description: 'Elasticsearch数据机构及存储'
keywords: Elasticsearch,全文检索,数据机构及存储
layout: post
status: public
title: Elasticsearch数据机构及存储
---

## 数据元机构（文档）  
1、在Elasticsearch中，文档(document)这个术语有着特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）  

2、一个文档必定包含三个元数据字段，_index,_type,_id  
_index	文档存储的地方,前文中索引的名称，也就是我们关系型数据库中的数据库名称，这个名字必须是全部小写，不能以下划线开头，不能包含逗号  
_type	文档代表的对象的类，名字可以是大写或小写，不能包含下划线或逗号  
_id	文档的唯一标识，当创建一个文档，你可以自定义_id，也可以让Elasticsearch帮你自动生成  


## _id  
1、id分两种自定义id,及自生成id  
第一种直接命令：

    PUT /website/blog/123
    {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
    }

自生成id:

    POST /website/blog/
    {
      "title": "My second blog entry",
      "text":  "Still trying this out...",
      "date":  "2014/01/01"
    }
    生成数据：
    {
       "_index":    "website",
       "_type":     "blog",
       "_id":       "wM0OSFhDQXGZAWDf0-drSA",
       "_version":  1,
       "created":   true
    }

自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 UUIDs。


    
