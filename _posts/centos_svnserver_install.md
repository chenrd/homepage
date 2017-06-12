---
categories: SVN
date: 2017-06-16 11:54:00
description: 'linux-centos下安装配置svn服务器'
keywords: svn
layout: post
status: public
title: subversion 安装配置
---

## 安装开始
>1、官方的安装指南：http://subversion.apache.org/packages.html#centos<br/>
    
>2、从文档中可以看出来最简单的一种安装方式：yum install subversion，不过直接安装不一定是最新的版本，如果要安装其他版本，这里提供两种办法：<br/>
>>一、就是通过RPM命令替换subversion包，再执行yum install subversion安装<br/>
>>二、上面的链接可以下载到完整的安装包，通过命令make && make install安装<br/>
    
>3、安装完成之后用svn --version命令查看是否正确安装<br/>
    
>4、新建一个自己的svn工作目录，mkdir /home/svnhome/,然后进入这个目录cd /home/svnhome/<br/>
    
>6、开始创建一个svn仓库：svnadmin create project，project是你的仓库名称，自己命名<br/>
>>创建仓库之后会在/home/svnhome/创建文件夹project,project文件夹中会有仓库的初始化文件<br/>
        
>7、cd /home/svnhome/project/conf/;进入到仓库的配置文件初始化的三个文件：authz,passwd,svnserve.conf<br/>

>8、vim svnservi.conf;修改svn仓库的配置文件<br/>
>>anon-access = none<br/>
>>auth-access = write<br/>
>>password-db = passwd<br/>
>>authz-db = authz<br/>
>>realm = project<br/>

>9、修改authz,passwd两个文件，配置用户及权限，文件里面已经注解有配置说明这里就不添加配置<br/>

>10、启动这个仓库:svnserve -d -r /svn/svnhome/project<br/>

>11、测试这个链接:svn list svn://192.168.1.xx/project 
