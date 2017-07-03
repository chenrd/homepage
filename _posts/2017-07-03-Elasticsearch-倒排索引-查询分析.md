---
categories: Elasticsearch
date: 2017-07-03 18:10
description: 'Elasticsearch倒排索引-查询分析器-确切值与全文检索'
keywords: Elasticsearch,索引,查询分析器
layout: post
status: public
title: Elasticsearch倒排索引-查询分析器
---

## 全文检索，确切值查询  
1、GET /_search?q=date:2014-09-15确切值查询  
2、GET /_search?q=2014-09-15全文检索  

## 查询分析器，分析器的几个步骤  
1、第一步：字符过滤器，字符过滤器能够去除HTML标记，或者转换"&"为"and"  
2、第二部：分词器（标准分析器、简单分析器、空格分析器、语言分析器）  

    测试下面的句子不同的分析器，分析之后的词组
    "Set the shape to semi-transparent by calling set_trans(5)"
    set, the, shape, to, semi, transparent, by, calling, set_trans, 5
    set, the, shape, to, semi, transparent, by, calling, set, trans
    Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
    set, shape, semi, transpar, call, set_tran, 5
    
3、确切值查询将不会使用分析器  
4、测试分析器：GET /_analyze?analyzer=standard&text=Text to analyze  
5、字段类型：GET /gb/_mapping/tweet  

6、复合类型，内部对象，及数组  
Elasticsearch的索引会对内部文件进行扁平化处理，{name: "test", child:{name: "小孩", age: 2}}  
-> {name: "test", "child.name":"小孩", "child.age": 2}  
数组：{name: "test", child:[{name: "小孩1", age: 2}, {name: "小孩2", age: 1}]}  
-> {name: "test", "child.name": ["小孩1", "小孩2"], "child.age":[2, 1]}  


