---
categories: logstash
date: 2017-08-15 22:51
description: 'logstash日志分析'
keywords: logstash,ELK,日志分析
layout: post
status: public
title: logstash日志输出
---

最近一段时间在研究日志分析系统，现在非常流行的一套日志分析系统ELK，网上查询了一些资料，现在把自己的学习分享出来

首先是logstash，日志处理机制。简单的描述：就是通过监控文件或者端口获取日志（input）,定义日志过滤规则（filter）过滤转化成需要的形式，最后输出到指定的文件或者其他查件中（output）

### 下载安装
到官方下载https://www.elastic.co/downloads/logstash 压缩包，logstash依赖jdk环境

解压压缩包之后就成功了，非常简单。

logstash命令，启动命令：

```
$ ./logstash -f nginx_log.conf 
```

> 重点
> 一个启动文件里面可以与多个input,只能有一个output。如果要针对不同的input输出到不同的地方，可以通过input的type属性来判断输出
>
> logstash启动的时候如果有多个配置文件，logstash会自动把多个文件拼接成一个文件启动

上面的命令是守护进程模式启动，退出之后进程就结束了，想要长期运行可以用下面的命令：

```
$ nohup ./logstash -f nginx_log.conf > /dev/null 2>&1 &
/dev/null 可以是自己制定的目录
```


logstash-plugin命令，插件命令

```
Usage:
    bin/logstash-plugin [OPTIONS] SUBCOMMAND [ARG] ...

Parameters:
    SUBCOMMAND                    subcommand
    [ARG] ...                     subcommand arguments

Subcommands:
    install                       Install a plugin
    uninstall                     Uninstall a plugin
    update                        Install a plugin
    list                          List all installed plugins

Options:
    -h, --help                    print help
```

首先，你可以通过 bin/logstash-plugin list 查看本机现在有多少插件可用。(其实就在 vendor/bundle/jruby/1.9/gems/ 目录下)

然后，假如你看到 https://www.elastic.co/downloads/logstash 下新发布了一个 logstash-output-webhdfs 模块(当然目前还没有)。打算试试，就只需要运行：

```
bin/logstash-plugin install logstash-output-webhdfs
```

问题记录：启动logstash的时候报错：

[2017-08-16T22:40:14,183][ERROR][logstash.agent           ] Cannot create pipeline {:reason=>"You are using a plugin that doesn't support workers but have set the workers value explicitly! This plugin uses the single and doesn't need this option"}

原因就是启动的选定的文件里面使用的插件没有安装。

### 公共属性

|字段         |描述                 |
|:----------|:--------------------|
|add_filed  |向事件添加一个字段      |
|codec      |编解码器              |
|enable_metric|                   |
|id           |强烈推荐设置此值，当有多个相同的插件时，可以区分|
|tags       |可以帮助稍后处理       |
|type       |                     |

### input

&nbsp;&nbsp;&nbsp;&nbsp;input 下面有非常多的插件，常用的有：file,syslog,tcp,redis,log4j等，详细的插件列表及插件使用方法可以查看[官方文档](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)

&nbsp;&nbsp;&nbsp;&nbsp;这里不对所有的插件详细讲解，只是把用到的插件记录一下，第一个log4j

#### log4j

&nbsp;&nbsp;&nbsp;&nbsp;

|字段       |描述                 |
|:----------|:--------------------|
|mode       |client模式会主动请求java应用端获取日志，server模式会接收java应用发过来的日志 |
|host       |启动logstash服务的主机，即localhost |
|port       |端口号              |
|tags       |可以帮助稍后处理       |
|type       |                     |


### filter

[fiter-官方](https://www.elastic.co/guide/en/logstash/5.5/filter-plugins.html)
