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

