---
categories: spring
date: 2017-10-24 18:08
description: 'spring-cloud-config记录一些注意事项，加使用实例'
keywords: spring,spring boot
layout: post
status: public
title: spring-boot
---

project: spring-boot-autoconfigure spring boot 自动化配置的入口，application.properties

project: spring-boot-autoconfigure : org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration 自动加载数据源的入口

spring boot 依赖三大容器启动：tomcat, jetty, undertow. EmbeddedServletContainer
