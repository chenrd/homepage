---
categories: spring
date: 2017-10-24 18:08
description: 'spring-cloud-config记录一些注意事项，加使用实例'
keywords: spring,spring cloud,cloud,spring cloud config
layout: post
status: public
title: spring-cloud-config 使用注意事项
---

### 注意事项

拉取资源

```
curl localhost:1201/config-server/development
```

spring-cloud-config支持多种方式作为属性仓库：git(推荐),svn,native(本机),Vault

#### 本地克隆仓库

- 基于（git,svn）的配置系统，会克隆远程文件到本地系统。linux下的默认路径/tmp/config-repo-<randomid>，可以通过修改spring.cloud.config.server.git.basedir或者spring.cloud.config.server.svn.basedir设置克隆目录

- 某些情况，比如：本地文件被手动修改时，将使远程克隆更新失败。可以通过配置属性来强制替换覆盖本地文件：force-pull:true

- cloneOnStart配置启动的时候clone远程资源，默认false, 第一次请求的时候clone远程资源

#### 使用本机配置文件

- spring.profiles.active = native

- spring.cloud.config.server.native.searchLocations指定文件目录

#### 属性覆盖

```
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```

#### 安全 属性加密

> 先决条件：要使用加密和解密功能，您需要在JVM中安装全面的JCE（默认情况下不存在）。您可以从Oracle下载“Java加密扩展（JCE）无限强度管理策略文件”，并按照安装说明（实际上将JRE lib / security目录中的2个策略文件替换为您下载的文件）。

> jce下载，java官方网站搜索jce下载对应jdk版本的文件，替换到jer/lib/security/包下面

> 使用方式 spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ

- 服务端默认配置了两个端点用来加解密/encrypt和/decrypt

加密：

```
$ curl localhost:8888/encrypt --data-urlencode mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

解密：

```
$ curl localhost:8888/decrypt --data-urlencode 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

另外一种加解密方式：安装了Spring Cloud CLI扩展，通过插件加解密

##### 对称加密与非对称加密

- 要配置对称密钥，您只需要将encrypt.key设置为一个秘密字符串（或使用环境变量ENCRYPT_KEY将其从纯文本配置文件中删除）

要配置非对称密钥，您可以将密钥设置为PEM编码的文本值（encrypt.key），也可以通过密钥库设置密钥（例如由JDK附带的keytool实用程序创建）。密钥库属性为encrypt.keyStore.*，*等于

- location（a Resource位置），

- password（解锁密钥库）和

- alias（以识别商店中使用的密钥）。

要创建一个密钥库进行测试，您可以执行以下操作：

```
keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```

将server.jks文件放在类路径（例如）中，然后在您的application.yml中配置服务器：

```
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```

#### 配置实例

实例1: 

search-paths通配spring.application.name

server 配置:

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/chenrd/novel-config-repo.git
          username: 53****@qq.com
          password: ***
          search-paths: '{application}'
```

client 配置:

```
spring:
  application:
    name: config-client
  cloud:
    config:
      uri: http://localhost:1201/
      profile: dev
      label: master
```

- 在git资源上新建目录config-client，在目录下新建application-dev.properties文件

- $ curl localhost:1201/config-client/dev 可以读取到文件


实例2:

模式匹配和多个存储库

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```

实例3：

多个配置文件

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - */development
                - */staging
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - */qa
                - */production
              uri: https://github.com/staging/config-repo
```




