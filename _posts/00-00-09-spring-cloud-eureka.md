---
categories: spring
date: 2017-10-24 18:08
description: 'spring-cloud 服务注册与发现'
keywords: spring,spring cloud,cloud,spring cloud eureka
layout: post
status: public
title: spring-cloud 服务注册与发现
---

### 概念

学习服务注册与发现首先需要弄清楚几个概念:

- 分布式服务发现服务器，目前主流的：zookeeper, eureka

- CAP(C- 数据一致性；A-服务可用性；P-服务对网络分区故障的容错性，这三个特性在任何分布式系统中不能同时满足，最多同时满足两个)

- ZooKeeper是个 CP的，eureka是AP

zookeeper与eureka比较：[点击链接](http://blog.csdn.net/jenny8080/article/details/52448403)

### eureka服务

- spring cloud 服务注册发现服务器eureka需要继承项目部署

- 类似dubble框架的注册服务器zookeeper，不同的是：zookeeper下载启动独立的安装包就可以了

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.M2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.0.M3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

启动函数Main:

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(EurekaServerApplication.class).web(true).run(args);
    }
}

```

配置文件bootstrap.yml

```
spring:
  application:
    name: novel-eureka-server
  cloud:
    config:
      uri: http://user:secret@localhost:1201/
      profile: development
      label: master
      
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

server:
  port: 8080

```


### 注册与消费

