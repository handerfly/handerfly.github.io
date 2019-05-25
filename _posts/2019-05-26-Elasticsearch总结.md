---
layout:     post
title:      Elasticsearch总结
date:       2019-05-26
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: Elasticsearch
tags:
    - Elasticsearch
    - ELK
---

# 检查jdk
```
java -version
```
Centos7一般都会带有自己的openjdk,我们一般都回用oracle的jdk,所以要卸载

步骤一：查询系统是否以安装jdk
```
rpm -qa|grep java
rpm -qa|grep jdk
rpm -qa|grep gcj 
```
步骤二：卸载已安装的jdk
```
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.131-11.b12.el7.x86_64
rpm -e --nodeps java-1.7.0-openjdk-1.7.0.141-2.6.10.5.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.131-11.b12.el7.x86_64
rpm -e --nodeps java-1.7.0-openjdk-headless-1.8.0.131-11.b12.el7.x86_64

# yum remove java*
```
步骤三：验证一下是还有jdk
```
rpm -qa|grep java
java -version
```
# 安装JDK8 
步骤一：下载linux版本的jdk并上传到linux上
步骤二：解压jdk包
```
tar -zxvf dk-8u144-linux-x64.tar.gz
```
步骤三：编辑/etc/profile文件，配置环境变量
```
export JAVA_HOME=/home/look/dev-software/jdk1.8.0_144
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
步骤四：生效profile
```
source /etc/profile
```

# elasticsearch安装
```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.0-linux-x86_64.tar.gz
tar -xvf elasticsearch-7.1.0-linux-x86_64.tar.gz
cd elasticsearch-7.1.0/bin
./elasticsearch  # ./elasticsearch -d 后台运行
```

> Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
这是因为JDK版本过高导致的，本机使用JDK12。

> Exception in thread "main" java.nio.file.AccessDeniedException: /usr/local/elasticsearch-7.1.0/config/jvm.options
elasticsearch用户没有该文件夹的权限，执行命令
chown -R es:es /usr/local/elasticsearch/

[jdk下载地址](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
[elasticsearch下载地址](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html)


