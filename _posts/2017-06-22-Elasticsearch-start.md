---
categories: Elasticsearch
date: 2017-06-22 14:54
description: '全文检索数据库Elasticsearch开始学习'
keywords: Elasticsearch,全文检索
layout: post
status: public
title: Elasticsearch详细讲解从安装开始
---
### 安装

&nbsp;&nbsp;&nbsp;&nbsp;说明：这里是对Elasticsearch学习的记录总结，[中文文档](https://es.xiaoleilu.com/010_Intro/05_What_is_it.html)

- 安装：下载地址：http://www.elasticsearch.org/download/  

- 最新版本5.4.zip包下载过来之后，要求jdk版本要1.8以上，附上JDK各版本下载地址http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html  

- 解压之后进入目录启动：./bin/elasticsearch，后台启动./bin/elasticsearch -d  

- 配置文件在config/elasticsearch.yml  

&nbsp;&nbsp;&nbsp;&nbsp;第一个命令：测试时候安装成功
```
$ curl 'http://localhost:9200/?pretty'
{
  "name" : "02ZKYGq",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "D4zf4nY1Sq66WQ1LImY5Ug",
  "version" : {
    "number" : "5.5.2",
    "build_hash" : "b2f0c09",
    "build_date" : "2017-08-14T12:33:14.154Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

### RESTful API

&nbsp;&nbsp;&nbsp;&nbsp;其他所有程序语言都可以使用RESTful API，通过9200端口的与Elasticsearch进行通信，你可以使用你喜欢的WEB客户端，事实上，如你所见，你甚至可以通过curl命令与Elasticsearch通信

> NOTE
>
> Elasticsearch官方提供了多种程序语言的客户端——Groovy，Javascript， .NET，PHP，Perl，Python，以及 Ruby——还有很多由社区提供的客户端和插件，所有这些可以在文档中找到。

&nbsp;&nbsp;&nbsp;&nbsp;向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

- VERB HTTP方法：GET, POST, PUT, HEAD, DELETE
- PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
- HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
- PORT Elasticsearch HTTP服务所在的端口，默认为9200
- PATH API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
- QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据
- BODY 一个JSON格式的请求主体（如果请求需要的话）

### elasticsearch.yml

&nbsp;&nbsp;&nbsp;&nbsp;配置文件elasticsearch.yml

|属性名称                |描述            |
|:----------------------|:--------------|
|cluster.name           |集群名称，在用一个网络环境下，通过该字段自动组成集群 |
|node.name              |节点名称         |
|node.attr.rack         |节点自定义属性    |
|path.data              |/path/to/data存储数据的路径,多个用逗号分隔|
|path.logs              |存储日志的路径    |
|bootstrap.memory_lock  |true启动时锁定内存，确保elasticsearch的堆内存为可用内存的一半    |
|network.host           |192.168.0.1,绑定ip   |
|http.port              |9200，启动端口      |
|discovery.zen.ping.unicast.hosts|，["127.0.0.1", "[::1]"]启动发现新节点的主机地址 |
|discovery.zen.minimum_master_nodes|                        |
|gateway.recover_after_nodes    |3，重启集群恢复，要先在多少个节点启动之后进行|
|action.destructive_requires_name|true,删除节点的时候要显示名称  |

### 集群、分片、索引、文档

下一个重要命令：集群健康检查

|颜色         |意义            |
|:-----------|:--------------|
|green       |所有主要分片和复制分片都可用    |
|yellow      |所有主要分片可用，但不是所有复制分片都可用 |
|red         |不是所有的主要分片都可用      |

```
GET /_cluster/health
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

关于分片：

- Elasticsearch数据保存在分片中，但是分片不与客户端交互，与客户端交互的是索引，每一个分片都是一个搜索引擎

- 分片与索引关联，分片分主分片与副分片，创建索引的时候就会创建分片，主分片创建之后个数固定，副分片个数可以修改，主分片（默认5个）

- 每个主分片都有副分片，主分片与副分片，分开存储在集群中，保证集群中的任意一个节点挂掉，集群数据都是完整的，可以正常进行

- 节点挂掉之后，如果有主分片死亡，那么在集群的其他节点会选择一个副分片升级成主分片

下面几张图，来说明一下（添加节点）、（添加副分片个数）、（节点挂掉）的时候节点的分片变化：

![posts_elas](http://chenrd.me/images/posts/elas_0203.png)
![posts_elas](http://chenrd.me/images/posts/elas_0204.png)
![posts_elas](http://chenrd.me/images/posts/elas_0205.png)
![posts_elas](http://chenrd.me/images/posts/elas_0206.png)

&nbsp;&nbsp;&nbsp;&nbsp;上面讲述了索引与分片的关系，那么Elasticsearch又是如何确定文档改存放在哪个分片上面呢？

```
shard = hash(routing) % number_of_primary_shards
```

&nbsp;&nbsp;&nbsp;&nbsp;通过上面这个简单的算法，确定文档存放在哪个分片，routing可以自定义，默认是文档_id


### 数据结构

一、传统数据库与elasticsearch概率关系对比  
    
    Relational DB -> Databases -> Tables -> Rows -> Columns
    Elasticsearch -> Indices   -> Types  -> Documents -> Fields

&nbsp;&nbsp;&nbsp;&nbsp;Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。

二、索引与分片的关系  
&nbsp;&nbsp;&nbsp;&nbsp;PUT /testindex,命令回去创建一个索引：testindex,创建一个索引默认会创建5个主分片，文档保存在分片当中，客户端不直接与分片交互，索引建立与分片的联系客户端通过索引检索文档  

    PUT /blogs
    {
       "settings" : {
          "number_of_shards" : 3,
          "number_of_replicas" : 1
          }
      }  

创建一个索引，指定3个主分片，1个副分片（每个主分片都有1个副分片），主分片在创建索引的时候就已经固定无法调整，副分片的数量可以随时调整。



### 数据元机构（文档）  
&nbsp;&nbsp;&nbsp;&nbsp;在Elasticsearch中，文档(document)这个术语有着特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）  

2、一个文档必定包含三个元数据字段，_index,_type,_id  

- _index	文档存储的地方,前文中索引的名称，也就是我们关系型数据库中的数据库名称，这个名字必须是全部小写，不能以下划线开头，不能包含逗号  

- _type	文档代表的对象的类，名字可以是大写或小写，不能包含下划线或逗号  

- _id	文档的唯一标识，当创建一个文档，你可以自定义_id，也可以让Elasticsearch帮你自动生成  


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

&nbsp;&nbsp;&nbsp;&nbsp;自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique identifiers, 或者叫 UUIDs。

### 数据类型及映射

Elasticsearch支持以下简单字段类型：

|类型            |表示的数据类型             |
|:---------------|:-----------------------|
|String	        |string             |
|Whole number	|byte, short, integer, long|
|Floating point	|float, double      |
|Boolean	    |boolean        |
|Date	        |date           |

当你索引一个包含新字段的文档——一个之前没有的字段——Elasticsearch将使用动态映射猜测字段类型，这类型来自于JSON的基本数据类型，使用以下规则：

|JSON type	        |Field type
|:------------------|:-----------------------|
|Boolean:true or false	           |"boolean"|
|Whole number:123	      |"long"           |
|Floating point:123.45	    |"double"         |
|String, valid date: "2014-09-15"   |"date"  |
|String: "foo bar"  |"string"                |

重要命令，查看类型映射：

```
GET /gb/_mapping/tweet  // db是索引，tweet是文档
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;当你向索引中加入数据的时候Elasticsearch会根据你入库的数据来猜测数据类型，自动完成类型映射。如果不想Elasticsearch来自动完成映射可以通过自定义实现：

#### 自定义字段映射

&nbsp;&nbsp;&nbsp;&nbsp;虽然大多数情况下基本数据类型已经能够满足，但你也会经常需要自定义一些特殊类型（fields），特别是字符串字段类型。 自定义类型可以使你完成以下几点：

- 区分全文（full text）字符串字段和准确字符串字段（译者注：就是分词与不分词，全文的一般要分词，准确的就不需要分词，比如『中国』这个词。全文会分成『中』和『国』，但作为一个国家标识的时候我们是不需要分词的，所以它就应该是一个准确的字符串字段）。

- 使用特定语言的分析器（译者注：例如中文、英文、阿拉伯语，不同文字的断字、断词方式的差异）

- 优化部分匹配字段

- 指定自定义日期格式（译者注：这个比较好理解,例如英文的 Feb,12,2016 和 中文的 2016年2月12日）

#### index

|值	          |解释         |
|:------------|:------------|
|analyzed	  |首先分析这个字符串，然后索引。换言之，以全文形式索引此字段。|
|not_analyzed |索引这个字段，使之可以被搜索，但是索引内容和指定值一样。不分析此字段。|
|no	          |不索引这个字段。这个字段不能为搜索到。|

#### 分析器
&nbsp;&nbsp;&nbsp;&nbsp;标准分析器、简单分析器、空格分析器、语言分析器，详细请看[下一小节](#查询分析器，分析器的几个步骤)

添加索引的时候指定映射：

```
PUT /gb <1>
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```

#### 更新映射

> 重要
>
>&nbsp;&nbsp;&nbsp;&nbsp;你可以向已有映射中增加字段，但你不能修改它。如果一个字段在映射中已经存在，这可能意味着那个字段的数据已经被索引。如果你改变了字段映射，那已经被索引的数据将错误并且不能被正确的搜索到。
>
>&nbsp;&nbsp;&nbsp;&nbsp;类似常用命令章节中：字符串排序

```
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```

#### 模板

&nbsp;&nbsp;&nbsp;&nbsp;索引可使用预定义的模板进行创建,这个模板称作Index templates。模板设置包括settings和mappings，通过模式匹配的方式使得多个索引重用一个模板，例如：

查看定义的模板：

```
curl -XGET localhost:9200/_template/template_1 
```

删除模板：

```
curl -XDELETE localhost:9200/_template/template_1  
```

下面定义的模板template_1将对用te开头的新索引都是有效。

```
curl -XPUT localhost:9200/_template/template_1 -d '
{
    "template" : "te*",
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : {"enabled" : false }
        }
    }
}
'
```

模板中也可以包含别别名的定义，如下：

```
curl -XPUT localhost:9200/_template/template_1 -d '
{
    "template" : "te*",
    "settings" : {
        "number_of_shards" : 1
    },
    "aliases" : {
        "alias1" : {},
        "alias2" : {
            "filter" : {
                "term" :{"user" : "kimchy" }
            },
            "routing" :"kimchy"
        },
        "{index}-alias" : {} 
    }
}
```

多个索引模板：

&nbsp;&nbsp;&nbsp;&nbsp;当存在多个索引模板时并且某个索引两者都匹配时，settings和mpapings将合成一个配置应用在这个索引上。合并的顺序可由索引模板的order属性来控制。

下面order为1的配置将覆盖order为0的配置，最终索引的配置source的enabled为true。

```
curl -XPUT localhost:9200/_template/template_1 -d '
{
    "template" : "*",
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : {"enabled" : false }
        }
    }
}
'
==================================================================
curl -XPUT localhost:9200/_template/template_2 -d '
{
    "template" : "te*",
    "order" : 1,
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : {"enabled" : true }
        }
    }
}
'
```

&nbsp;&nbsp;&nbsp;&nbsp;对于Elasticsearch的搜索查询来说，数据大致可以分为两类：确切值与全文文本

1、GET /_search?q=date:2014-09-15确切值查询

2、GET /_search?q=2014-09-15全文检索  

### 查询分析器，分析器的几个步骤

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

- Elasticsearch的索引会对内部文件进行扁平化处理，{name: "test", child:{name: "小孩", age: 2}} -> {name: "test", "child.name":"小孩", "child.age": 2}

- 数组：{name: "test", child:[{name: "小孩1", age: 2}, {name: "小孩2", age: 1}]} -> {name: "test", "child.name": ["小孩1", "小孩2"], "child.age":[2, 1]}



### 性能问题记录

- 警告： 对 analyzed 字段进行强制排序会消耗大量内存

