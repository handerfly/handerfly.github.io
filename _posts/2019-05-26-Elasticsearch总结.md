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
curl -XPUT 'localhost:9200/customer?pretty'
# 或者
curl -X PUT "localhost:9200/customer" -H 'Content-Type: application/json' -d'
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
# 索引并查询一个文档 
```
curl -X PUT "localhost:9200/customer/user/1?pretty" -H 'Content-Type: application/json' -d'{"name": "John Doe"}'
```
# 修改一个索引
```
curl -X PUT 'localhost:9200/customer/_settings' -H 'Content-type:Application/json' -d '{"index":{"number_of_replicas":2}}'
```

# 修改数据
在 Elasticsearch 中文档是 不可改变 的，不能修改它们.每当我们执行更新时，Elasticsearch就会删除旧文档，然后索引一个新的文档
```
curl -X POST "localhost:9200/customer/user/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'
```
使用script修改数据
```
[han@node1 ~]$ curl -X POST "localhost:9200/customer/user/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}
'
#返回如下内容
{
  "_index" : "customer",
  "_type" : "user",
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
curl -X DELETE "localhost:9200/customer?pretty"
```


# _bulk API批量执行
索引两个文档（ID 1 - John Doe 和 ID 2 - Jane Doe）
```
curl -X POST "localhost:9200/customer/user/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'
```

# 批量插入数据

```
[han@node1 ~]$ cat accounts.json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
{"index":{"_id":"13"}}
{"account_number":13,"balance":32838,"firstname":"Nanette","lastname":"Bates","age":28,"gender":"F","address":"789 Madison Street","employer":"Quility","email":"nanettebates@quility.com","city":"Nogal","state":"VA"}
{"index":{"_id":"18"}}
{"account_number":18,"balance":4180,"firstname":"Dale","lastname":"Adams","age":33,"gender":"M","address":"467 Hutchinson Court","employer":"Boink","email":"daleadams@boink.com","city":"Orick","state":"MD"}

[han@node1 ~]$ curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
```

# The Search API







































































































# elasticsearch中 refresh 和flush区别
elasticsearch中有两个比较重要的操作：refresh 和 flush      
refresh操作    
当我们向ES发送请求的时候，我们发现es貌似可以在我们发请求的同时进行搜索。而这个实时建索引并可以被搜索的过程实际上是一次es 索引提交（commit）的过程，如果这个提交的过程直接将数据写入磁盘（fsync）必然会影响性能，所以es中设计了一种机制，即：先将index-buffer中文档（document）解析完成的segment写到filesystem cache之中，这样避免了比较损耗性能io操作，又可以使document可以被搜索。以上从index-buffer中取数据到filesystem cache中的过程叫做refresh。
 
 

refresh操作可以通过API设置：   
```
POST /index/_settings
{“refresh_interval”: “10s”}
```
当我们进行大规模的创建索引操作的时候，最好将将refresh关闭。   
```
POST /index/_settings
{“refresh_interval”: “-1″}
```
es默认的refresh间隔时间是1s，这也是为什么ES可以进行近乎实时的搜索。   
 
flush操作与translog   
我们可能已经意识到如果数据在filesystem cache之中是很有可能在意外的故障中丢失。这个时候就需要一种机制，可以将对es的操作记录下来，来确保当出现故障的时候，保留在filesystem的数据不会丢失，并在重启的时候可以从这个记录中将数据恢复过来。elasticsearch提供了translog来记录这些操作。
当向elasticsearch发送创建document索引请求的时候，document数据会先进入到index buffer之后，与此同时会将操作记录在translog之中，当发生refresh时（数据从index buffer中进入filesystem cache的过程）translog中的操作记录并不会被清除，而是当数据从filesystem cache中被写入磁盘之后才会将translog中清空。而从filesystem cache写入磁盘的过程就是flush。可能有点晕，我画了一个图帮大家理解这个过程：

![refreshflush](https://raw.githubusercontent.com/handerfly/handerfly.github.io/master/img/refresh.jpg)
               

总结一下translog的功能：   
1.保证在filesystem cache中的数据不会因为elasticsearch重启或是发生意外故障的时候丢失。   
2.当系统重启时会从translog中恢复之前记录的操作。   
3.当对elasticsearch进行CRUD操作的时候，会先到translog之中进行查找，因为tranlog之中保存的是最新的数据。   
4.translog的清除时间时进行flush操作之后（将数据从filesystem cache刷入disk之中）。   
 
 
再总结一下flush操作的时间点：   
1.es的各个shard会每个30分钟进行一次flush操作。   
2.当translog的数据达到某个上限的时候会进行一次flush操作。   
 
 
有关于translog和flush的一些配置项：   
index.translog.flush_threshold_ops:当发生多少次操作时进行一次flush。默认是 unlimited。   
index.translog.flush_threshold_size:当translog的大小达到此值时会进行一次flush操作。默认是512mb。   
index.translog.flush_threshold_period:在指定的时间间隔内如果没有进行flush操作，会进行一次强制flush操作。默认是30m。   
index.translog.interval:多少时间间隔内会检查一次translog，来进行一次flush操作。es会随机的在这个值到这个值的2倍大小之间进行一次操作，默认是5s。   



[jdk下载地址](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
[elasticsearch下载地址](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html)
[elasticsearch常用插件](https://www.cnblogs.com/ZJ199012/p/6094083.html)
[CentOS 7.X 下安装ElasticSearch-Head插件](https://blog.csdn.net/xzwspy/article/details/78386415)


