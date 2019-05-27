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
或者下载rpm包安装
```
 直接将下载好的jdk-8u151-linux-x64.rpm 安装包 ;上传到自己创建好的JAVA文件下；
 cd 命令进入到JAVA文件下使用rpm 命令进行安装 
 rpm -ivh jdk-8u131-linux-x64.rpm 安装完成后执行 
 java -version 命令查看安装是否成功
   3.1.3 查看安装目录命令，
      命令一：which java 
      命令二：ls -lrt /usr/bin/java
      命令三：ls -lrt /etc/alternatives/java
      最后将会得出这样的目录 /usr/java/jdk1.8.0_151/jre/bin/java

   3.1.4 配置环境变量，执行命令 vi /etc/profile；然后进入编辑模式，在文件的最后添加下面的配置，如图

      JAVA_HOME=/usr/javajdk1.8.0_151
      JRE_HOME=/usr/java/jdk1.8.0_151/jre
      CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
      PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
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
注意:elasticsearch不能用root运行
> Exception in thread "main" java.nio.file.AccessDeniedException: /usr/local/elasticsearch-7.1.0/config/jvm.options
elasticsearch用户没有该文件夹的权限，执行命令
chown -R es:es /usr/local/elasticsearch/

>{"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}
curl -H "Content-Type: application/json" -XPUT  加上-H参数

# 配置
```
cluster.name: elasticsearch_production
node.name: elasticsearch_005_data

# Path to directory where to store the data (separate multiple locations by comma):
path.data: /path/to/data1,/path/to/data2 

# Path to log files:
path.logs: /path/to/logs

# Path to where plugins are installed:
path.plugins: /path/to/plugins

# 最小主节点数
# 方法1 编辑elasticsearch.yml
discovery.zen.minimum_master_nodes: 2

# 方法2 命令修改 (优先级更高)
PUT /_cluster/settings
{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}

# Elasticsearch集群的脑裂问题：
# 将master节点与data节点分离
# 指定该节点是否有资格被选举成为node（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
node.master: true 

# 指定该节点是否存储索引数据，默认为true。
node.data: false  

# 为了使新加入的节点快速确定master位置，可以将data节点的默认的master发现方式有multicast修改为unicast
# 设置是否打开多播发现节点，默认是true。
discovery.zen.ping.multicast.enabled: false
# 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
discovery.zen.ping.unicast.hosts: ["master1", "master2", "master3"]
discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]

# 默认值是3秒默认情况下，一个节点会认为，如果master节点在3秒之内没有应答，那么这个节点就是死掉了，而增加这个值，会增加节点等待响应的时间
discovery.zen.ping.timeout: 3s
# 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量，
discovery.zen.minimum_master_nodes: 1
```


# ES中常用的响应状态码    
200 OK - 一般表示操作成功
404 Not Found - 一般在查询时找不到文档是返回404找不到
201 Created - 一般创建文档成功时返回201已经创建
409 Conflict - 一般创建文档或者更新文档时失败返回冲突


# The REST API

# cat API
列出所有可用的 API
```
curl -XGET 'http://localhost:9200/_cat/'
```

要启用表头，添加 ?v 参数即可
对任意 API 添加 ?help 参数,显示所有可用指标
?h 参数来明确指定显示这些指标，多个指标用逗号隔开
?bytes=b  字节显示
```
[han@node1 ~]$curl -XGET 'http://localhost:9200/_cat/health'
[han@node1 ~]$curl -XGET 'http://localhost:9200/_cat/health?v'
[han@node1 ~]$ curl -XGET 'http://localhost:9200/_cat/nodes?help'
id                                 | id,nodeId                                   | unique node id                                                                                      
pid                                | p                                           | process id                                                                                          
ip                                 | i                                           | ip address   
...........
[han@node1 ~]$curl -XGET 'http://localhost:9200/_cat/nodes?v&h=ip,port,heapPercent,heapMax'
ip        port heapPercent heapMax
127.0.0.1 9300          26 990.7mb

[han@node1 ~]$ curl 'localhost:9200/_cat/indices?bytes=b' | sort -rnk8
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   304  100   304    0     0  13947      0 --:--:-- --:--:-- --:--:-- 14476
yellow open test2                loCO7Wj9S9m7ZWVp8_FNcQ 5 1 0 0  1415  1415
yellow open test1                1OTOB-bsSzSd3vPzzkQo_Q 1 1 0 0   283   283
green  open .kibana_task_manager k-ZXcWc0SceWjfg4tkLrUg 1 0 2 0 13141 13141
green  open .kibana_1            Uq2RcOz-QPikjfDLXrDEiA 1 0 5 0 19735 19735
[han@node1 ~]$
[han@node1 ~]$ curl 'localhost:9200/_cat/indices' | sort -rnk8
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   312  100   312    0     0  31553      0 --:--:-- --:--:-- --:--:-- 34666
yellow open test2                loCO7Wj9S9m7ZWVp8_FNcQ 5 1 0 0  1.3kb  1.3kb
yellow open test1                1OTOB-bsSzSd3vPzzkQo_Q 1 1 0 0   283b   283b
green  open .kibana_task_manager k-ZXcWc0SceWjfg4tkLrUg 1 0 2 0 12.8kb 12.8kb
green  open .kibana_1            Uq2RcOz-QPikjfDLXrDEiA 1 0 5 0 19.2kb 19.2kb

```

# 集群状态
```
[han@node1 ~]$curl -X GET "localhost:9200/_cluster/health"
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
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
status 字段是我们最关心的。
status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
green
所有的主分片和副本分片都正常运行。
yellow
所有的主分片都正常运行，但不是所有的副本分片都正常运行。
red
有主分片没能正常运行


# 创建索引
> 在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。
在新建文档的时候如果指定的索引不存在则会自动创建相应的索引        
```
curl -XPUT 'localhost:9200/blogs?pretty'
# 或者
curl -X PUT "localhost:9200/blogs" -H 'Content-Type: application/json' -d'
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
'
```

# 查询索引
```
curl -X GET "localhost:9200/_cat/indices?v"
```
# 索引并查询一个文档  index:blogs, type:mysql, id:1
```
curl -X PUT "localhost:9200/blogs/mysql/1?pretty" -H 'Content-Type: application/json' -d'{"name": "John Doe"}'
```

# 修改数据
在 Elasticsearch 中文档是 不可改变 的，不能修改它们.每当我们执行更新时，Elasticsearch就会删除旧文档，然后索引一个新的文档
```
curl -X POST "localhost:9200/blogs/mysql/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'
```
使用script修改数据
```
[han@node1 ~]$ curl -X POST "localhost:9200/blogs/mysql/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}
'
#返回如下内容
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
'
```
# 删除一个索引
```
curl -X DELETE "localhost:9200/blogs?pretty"
```




















































































































[jdk下载地址](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
[elasticsearch下载地址](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html)
[elasticsearch常用插件](https://www.cnblogs.com/ZJ199012/p/6094083.html)
[CentOS 7.X 下安装ElasticSearch-Head插件](https://blog.csdn.net/xzwspy/article/details/78386415)


