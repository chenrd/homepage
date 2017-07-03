---
categories: Elasticsearch
date: 2017-07-03 20:02
description: 'Elasticsearch搜索查询'
keywords: Elasticsearch,全文检索,command,命令,查询,搜索
layout: post
status: public
title: Elasticsearch搜索
---

## 空检索，下面的语句会放回引擎中的所有文档  
GET /_search<br/>
{}<br/>

GET /index_2014*/type1,type2/_search<br/>
{}<br/>
//所在索引(index)模糊匹配，对象（表格）多个一起查询<br/>

    {
      "from": 30,
      "size": 10
    }
    //分页参数
## 结构化查询  

    GET /_search
    {
        "query": {
            "match_all": {}
        }
    }
    
例子：  
    
    GET /_search
    {
        "query": {
            "match": {
                "tweet": "elasticsearch"
            }
        }
    }
    
多个子句：  

    {
        "bool": {
            "must":     { "match": { "tweet": "elasticsearch" }},
            "must_not": { "match": { "name":  "mary" }},
            "should":   { "match": { "tweet": "full text" }}
        }
    }
    
bool合并多个子句：  
    
    {
        "bool": {
            "must": { "match":      { "email": "business opportunity" }},
            "should": [
                 { "match":         { "starred": true }},
                 { "bool": {
                       "must":      { "folder": "inbox" }},
                       "must_not":  { "spam": true }}
                 }}
            ],
            "minimum_should_match": 1
        }
    }
    
## 最重要的查询过滤语句  
1、term 过滤，term主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed的字符串(未经分析的文本数据类型)  
 
    { "term": { "age":    26           }}
    { "term": { "date":   "2014-09-01" }}
    { "term": { "public": true         }}
    { "term": { "tag":    "full_text"  }}
    
2、terms过滤 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配：

    {
        "terms": {
            "tag": [ "search", "full_text", "nosql" ]
            }
    }
    
3、range 过滤，range过滤允许我们按照指定范围查找一批数据：  

   {
        "range": {
            "age": {
                "gte":  20,
                "lt":   30
            }
        }
    }
    
范围操作符包含：<br/>
gt :: 大于<br/>
gte:: 大于等于<br/>
lt :: 小于<br/>
lte:: 小于等于<br/>

4、exists 和 missing 过滤，exists 和 missing 过滤可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的IS_NULL条件，这两个过滤只是针对已经查出一批数据来，但是想区分出某个字段是否存在的时候使用。  

    {
        "exists":   {
            "field":    "title"
        }
    }

5、bool 过滤<br/>

  bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：<br/>
  must :: 多个查询条件的完全匹配,相当于 and。<br/>
  must_not :: 多个查询条件的相反匹配，相当于 not。<br/>
  should :: 至少有一个查询条件匹配, 相当于 or。<br/>
  这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：<br/>



