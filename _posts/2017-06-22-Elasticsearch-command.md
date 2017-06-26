---
categories: Elasticsearch
date: 2017-06-22 14:54
description: 'Elasticsearch命令'
keywords: Elasticsearch,全文检索,command,命令
layout: post
status: public
title: Elasticsearch常用命令记录
---

1、获取文档GET /website/blog/123?_source=title,text，查询文档且文档中的_source对象只显示title,text属性  
GET /website/blog/123/_source,查询文档只显示文档中的_source对象  

2、添加更新整个文档文档PUT /website/blog/123  
id不存在，添加文档，created:true，文档已经存在更新整个文档，created:false,_version自增

3、创建一个新文档，不确定自定义ID的文档是否存在情况下创建新的文档  
一：POST /website/blog/ 自生成ID  
二：PUT /website/blog/123?op_type=create  
三：PUT /website/blog/123/_create  
Elasticsearch将返回正常的元数据且响应状态码是201 Created，文档已经存在，Elasticsearch将返回409  

3、检查文档是否存在curl -i -XHEAD http://localhost:9200/website/blog/123,Elasticsearch将会返回200 OK状态如果你的文档存在：  
    
    HTTP/1.1 200 OK
    Content-Type: text/plain; charset=UTF-8
    Content-Length: 0

4、删除文档DELETE /website/blog/123  
Elasticsearch将返回200 OK状态码和以下响应体。注意_version数字已经增加了。  
    
    {
      "found" :    true,
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "123",
      "_version" : 3
    }

5、检索多个文档，合并请求到一次检索:mget. 
    
    POST /_mget
    {...}
    //不在同一个_index中
    
    POST /website/blog/_mget
    {...}
    //在同一个_index中
    
    POST /website/blog/_mget
    {
       "ids" : [ "2", "1" ]
    }
    
6、操作批量：POST /_bulkd 批量请求性能：请求体的物理内存大小非常重要，一般来说一次批量请求控制在5-15MB之间比较好.  
    
    POST /_bulk
    { "create": { "_index": "website", "_type": "blog", "_id": "123" }}
    { "title":    "Cannot create - it already exists" }
    { "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
    { "title":    "But we can update it" } 
    //最后添加换行符  
    
7、搜索命令：_search  
GET /_search?q=mary 返回包含"mary"字符的所有文档，_all字段，查询不指定要查询的字段，会把所有的字段拼接成_all字段. 
/gb,us/user,tweet/_search. 
/g*,u*/_search  
GET /_all/tweet/_search?q=tweet:elasticsearch  
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary(+name:john +tweet:mary) 查找name字段中包含"john"和tweet字段包含"mary"的结果  

8、分页：GET /_search?size=5&form=10  






