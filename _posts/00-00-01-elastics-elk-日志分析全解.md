---
categories: ELK
date: 2017-08-26 12:00
description: 'ELK日志分析系统，详细分析记录'
keywords: elk,logstash,leasticsearch
layout: post
status: public
title: ELK日志分析系统
---

&nbsp;&nbsp;&nbsp;&nbsp;在分布式架构中，服务器数量上来之后日志的如果还是单台服务器管理会是非常繁琐的事情，吃力不讨好。现在网络上有比较多的日志分析框架，我这里选择ELK日志分析系统，一来是因为ELK分析系统是全开源的，本着支持开源的原则，二来ELK分析系统有大型互联网企业的使用经验（github等）。

&nbsp;&nbsp;&nbsp;&nbsp;现在开始先从结构上讲解一下，网络附图：

![posts_optional](http://chenrd.me/images/posts/elk_01.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;具体的部署结构看自己需求来定，看你需要监控的服务器有多少，如果你只有一，两台，那么你还是不要部署ELK了浪费资源，有需要的时候登录上去看一下就完成了。ELK本身是为了分布式结构，服务器集群日志管理的。

&nbsp;&nbsp;&nbsp;&nbsp;上面是闲话，接下来先看一下图片中Broker左边的(Shipper logstash)，Shipper logstash就是每个需要添加监控的服务器，每一台服务器部署一个logstash，这里的logstash用于监控日志的产出，称为：Shipper logstash。
Broker右边可以看成一个整体（包括Broker），Broker把所有的Shipper logstash产出的日志搜集起来，再通过Indexer Logstash输出到 ElasticSearch，最后在Kibana上展示。

&nbsp;&nbsp;&nbsp;&nbsp;补充上面：Broker角色可以是redis、kafka等其他中间件，官方推荐的是redis，文章下面用到的也是redis。主要是通过redis的订阅/发布及list队列来实现消息的传递，具体看下文实现。

### 部署我的第一个Shipper logstash角色
&nbsp;&nbsp;&nbsp;&nbsp;软件需要下载安装logstash：[下载地址](https://www.elastic.co/downloads/logstash)，文章使用的版本是5.5.2，直接下载压缩包.tar文件，解压开来就可以使用了。(备注：需要JDK环境)

&nbsp;&nbsp;&nbsp;&nbsp;第一个实例：监控nginx的访问日志access.log

```
$ tar -vxf logstash-5.5.2.tar
$ cd logstash-5.5.2/bin
$ ls
cpdump  ingest-convert.sh  logstash  logstash.bat  logstash.lib.sh  logstash-plugin  logstash-plugin.bat  ruby  setup.bat  system-install

//其中两个重要的命令：logstash（启动命令）,logstash-plugin（插件命令）
```

&nbsp;&nbsp;&nbsp;&nbsp;再来说一下logstash的启动，logstash默认守护进程方式启动，可以通过nohup方式+ 结尾加&方法启动

&nbsp;&nbsp;&nbsp;&nbsp;下面添加我的第一个logstash日志监控配置文件，nginx_log.conf：
```
input {
    file {
        path => ["/usr/local/nginx/logs/logstash.log"]
        type => "system"
        start_position => "end"
        discover_interval => 3600
        sincedb_path => "/usr/local/logstash-5.5.2/data/.sincedb"
        # 不想被监听的文件可以排除出去，这里跟 path 一样支持 glob 展开
        # exclude => []
        # 一个已经监听中的文件，如果超过这个值的时间内没有更新内容，就关闭监听它的文件句柄。默认是 3600 秒，即一小时
        # close_older => 7200
        # 在每次检查文件列表的时候，如果一个文件的最后修改时间超过这个值，就忽略这个文件。默认是 86400 秒，即一天
        # ignore_older => 86400

        # stat_interval,检查文件间隔时间默认1秒,start_position从文件哪个位置开始读取,sincedb_path保存监听文件状态的文件路径,incedb_write_interval
    }
}

output {
    redis {
        data_type => "channel" //相当于 list => BLPOP channel => SUBSCRIBE pattern_channel => PSUBSCRIBE， 对这个有疑问要去看redis的订阅/发布及list消息队列
        key => "logstash"
        host => "you.redis.host"
        port => 6379
        password => "you.redis.password"
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;nginx添加配置：
```
log_format logstash "$http_x_forwarded_for | $time_local | $request | $status | $body_bytes_sent | "
                "$request_body | $content_length | $http_referer | $http_user_agent | $nuid | "
                "$http_cookie | $remote_addr | $hostname | $upstream_addr | $upstream_response_time | $request_time";

server {
        listen          18081;
        server_name     localhost;
        location / {
                access_log logs/logstash.log logstash;
                return 200 'success';
        }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;这里简单说明一下logstash的工作原理：logstash监控文件主要由三个部分组成，input(输入)，filter(过滤)，output(输出)其中file,reids都是对应的插件，可以有其他的插件取代具体看官方中文文档：[官网文档](https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/get_start/index.html)

&nbsp;&nbsp;&nbsp;&nbsp;启动我的第一个Shipper logstash角色
```
nnohup ./logstash -f ../conf/nginx_log.conf >/dev/null 2>&1 &
```

&nbsp;&nbsp;&nbsp;&nbsp;查看启动日志，基本没问题，到这里就已经启动了第一个Shipper logstash实例了，到你的服务器集群中可以一台一台的部署上去，根据自己的需求选中不同的input插件（file,syslog,tcp等）监控不同的日志

&nbsp;nbsp;&nbsp;&nbsp;插件列表：

|插件                 |作用                          |
|:-------------------|:-----------------------------|
|ruby                |使用ruby脚本语言，非常强大，基本可以实现所有的逻辑了 |
|kv                  |通过：field_split把属性分割成key-value方式       |
|split               |把输入详细分割成多个输出消息                      |
|Mutate              |转换类型，logstash只有三种数据类型（"integer"，"float"，"string"）,字符串处理,字段处理|
|Metrics             |数值统计                     |
|json                |message是json格式，可以通过这个插件直接转成字段|
|grok                |正则表达式，比较消耗cpu，其他方式可以实现不建议用正则|
|date                |日期处理插件                  |
|dissect             |类似正则，为了解决正则的性能问题而开发的插件    |
|GeoIP               |GeoIP 是最常见的免费 IP 地址归类查询库  |
|elapsed             |                              |

### reids到indexer logstash - elasticSearch

&nbsp;&nbsp;&nbsp;&nbsp;与上面相同的安装方式，在自己的indexer服务器上面安装logstash。

&nbsp;&nbsp;&nbsp;&nbsp;从redis到indexer logstash，需要配置indexer logstash订阅到redis的对应渠道上面，下文中的：input -> redis -> key

&nbsp;&nbsp;&nbsp;&nbsp;中间还有一块fiter，入库的数据可能不符合要求这里需要用filter处理之后，统一格式再输出到elasticsearch中

&nbsp;&nbsp;&nbsp;&nbsp;具体logstash的用法请看作者的另外一篇文章：[logstash日志输出](http://chenrd.me/2017/08/15/elastic-logstash/)

> 注意elasticsearch的安装及详细用法可以查看文章[Elasticsearch详细讲解从安装开始](http://chenrd.me/2017/06/22/elastic-elasticsearch-start/)

```
input {
    redis {
	    data_type => "channel"
        key => "logstash-list"
        host => "121.40.92.116"
        port => 6379
        password => "@@leke_admin@@"
	    batch_count => 1
	    threads => 1
	    type => "nginxlog"
	    id => "nginxlog"
    }
    redis {
        data_type => "channel"
        key => "parentlog"
        host => "121.40.92.116"
        port => 6379
        password => "@@leke_admin@@"
        batch_count => 1
        threads => 1
        type => "webapilog"
	id => "webapilog"
    }
}
filter {
  if [id] == "nginxlog" {
    ruby {
	init => "@kname = ['http_x_forwarded_for','time_local','request','status','body_bytes_sent','request_body','content_length','http_referer','http_user_agent','http_cookie','remote_addr','hostname','upstream_addr','upstream_response_time','request_time']"
	code => "
	    new_event = LogStash::Event.new(Hash[@kname.zip(event.get('message').split('|'))])
	    new_event.remove('@timestamp')
	    event.append(new_event)
	"
    }
    if [request] {
	ruby {
	    init => "@kname = ['method', 'uri', 'verb']"
	    code => "
		new_event = LogStash::Event.new(Hash[@kname.zip(event.get('request').split(' '))])
                new_event.remove('@timestamp')
                event.append(new_event)
	    "
	    
	}
    	if [uri] {
            ruby {
                init => "@kname = ['url_path','url_args']"
                code => "
                    new_event = LogStash::Event.new(Hash[@kname.zip(event.get('uri').split('?'))])
                    new_event.remove('@timestamp')
                    event.append(new_event)
                "
            }
            kv {
                prefix => "url_"
                source => "url_args"
                field_split => "& "
                remove_field => [ "url_args","uri","request" ]
            }
        }
    }
    mutate {
        convert => [
            "body_bytes_sent" , "integer",
            "content_length", "integer",
            "upstream_response_time", "float",
            "request_time", "float"
        ]
    }
    date {
        match => [ "time_local", "dd/MMM/yyyy:hh:mm:ss Z" ]
        locale => "en"
    }
  }
    if [type] == "log" {
	dissect {
	    mapping => {
	    	"message" => "%{date} %{+date} %{level} %{line} %{msg}"
	    }	
	}
	date {
            match => [ "date", "dd/MMM/yyyy:HH:mm:ss Z" ]
            locale => "en"
    	}
    }
}
output {
    elasticsearch {
        hosts => ["localhost:9200"]
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
        document_type => "%{type}"
        flush_size => 20000
        idle_flush_time => 10
        sniffing => true
        template_overwrite => true
    }
}

```

### Kibana

配置不做详细解释，conf/kibana.yml文件对配置项有详细描述，开发情况下默认启动就可以了

Kibana默认的路径http://localhost:5601/

|菜单     |功能             |
|:--------|:---------------|
|Discover |显示所有字段，直观查看时间段内的所有日志|
|Visuablize|自定义图表       |
|Dashboard |图表             |
|Timelion  |图表             |
|dev Tools |开发工具，直接查询语句 |
|Management|配置              |

![主图](http://chenrd.me/images/posts/elk_02.jpg)

图中：Use event times to create index names \[DEPRECATED\] 按日期配置

正确的例子：\[logstash-log-\]YYYY.MM.DD

![Discover](http://chenrd.me/images/posts/elk_03.jpg)



### 问题记录

- elastics @timestamp 默认使用的是UTC时区，获取的时候比中国晚了8小时，elastics官方推荐的解决办法是在kibana上面切换时间显示来解决这个问题。

- 使用date插件来解决@timestamp时区不对的问题，date插件转换过后的属性值会替换掉原来的@timestamp







