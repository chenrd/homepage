---
categories: spring
date: 2017-06-29 11:31
description: 'spring-boot嵌入式tomcat，netty等web服务器'
keywords: spring,web,spring-boot
layout: post
status: public
title: spring-boot学习笔记
---

## 使用spring-boot过程中常用到的命令  

    $ mvn clean install
    $ mvn package
    $ java -jar target/myproject-0.0.1-SNAPSHOT.jar
    $ java -jar target/myproject-0.0.1-SNAPSHOT.jar --debug //打印出很多Java虚拟机的日志
    $ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -jar target/myproject-0.0.1-SNAPSHOT.jar
    //直接用java命令启动
    
    $ mvn spring-boot:run
    $ export MAVEN_OPTS=-Xmx1024m
    //用mvn的方式启动spring-boot
    
    $ java -jar myproject.jar --spring.config.name=myproject
    $ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
    
    //spring-boot 默认读取的是application*.properties文件，spring.config.name改掉之后读取的就是myproject*.properties
## spring-boot的资源文件加载及加载路径配置  
/META-INF/maven, /META-INF/resources ,/resources ,/static ,/public or /templates，这些文件下面的文件默认加载

You may want your application to be restarted or reloaded when you make changes to files that are not on the classpath. To do so, use the    
spring.devtools.restart.additional-paths property to configure additional paths to watch for changes. You can use the 
spring.devtools.restart.exclude property described above to control whether changes beneath the additional paths will trigger a full restart or just a live reload.
