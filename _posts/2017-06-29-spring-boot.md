---
categories: spring
date: 2017-06-29 11:31
description: 'spring-boot嵌入式tomcat，netty等web服务器'
keywords: spring,web,spring-boot
layout: post
status: public
title: spring-boot学习笔记
---

### 使用spring-boot过程中常用到的命令  

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
### spring-boot的资源文件加载及加载路径配置  
/META-INF/maven, /META-INF/resources ,/resources ,/static ,/public or /templates，这些文件下面的文件默认加载

You may want your application to be restarted or reloaded when you make changes to files that are not on the classpath. To do so, use the    
spring.devtools.restart.additional-paths property to configure additional paths to watch for changes. You can use the 
spring.devtools.restart.exclude property described above to control whether changes beneath the additional paths will trigger a full restart or just a live reload.  

### spring-boot 缓存记录  
spring-boot 支持的几个库使用缓存来提高性能。例如，模板引擎将缓存编译的模板，以避免重复解析模板文件  
此外，Spring MVC可以在服务静态资源时向响应添加HTTP缓存头  
虽然缓存在生产中非常有利，但在开发过程中可能会产生反效果，从而阻止您看到刚刚在应用程序中进行的更改。出于这样的原因spring-boot-devtools会禁用这些缓存选项。  
spring.thymeleaf.cache

### spring-boot-devtools开发工具类
spring-boot-devtools是加速开发的工具，代码修改之后自动重启，下面是spring-boot-devtools的一些配置属性：

spring.devtools.restart.enabled=true //是否开启自动重启
spring.devtools.restart.additional-paths= //添加自动重启监控文件地址，检测这个地址，那么这个地址的文件发生变化，触发重启
spring.devtools.restart.additional-exclude= //追述上一个配置，排除
spring.devtools.restart.exclude=META-INF/maven/**,META-INF/resources/**， //排除
spring.devtools.restart.poll-interval=1000 //时间单位毫秒，轮询登录资源路径的变化

spring-boot-devtools 加 springloaded实现热加载，类修改之后重新编译修改的这个类，自动重新加载这个类，不需要重启整个项目

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <!-- spring热部署-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.6.RELEASE</version>
        </dependency>
    </dependencies>
    <configuration>
        <mainClass>sample.tomcat.SampleTomcatApplication</mainClass>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

idea 没有开启自动编译的话，需要收到Build -> compile 当前类...
开启自动编译的方法查看：wiki - idea/mac 

#### 自定义servlet

> 官方解释 
> A @Bean of type Servlet or ServletRegistrationBean installs that bean in the container as if it were a <servlet/> and <servlet-mapping/> in web.xml.



#### 自定义servlet-filter

