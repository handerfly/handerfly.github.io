---
layout:     post
title:      Elasticsearch配置
date:       2019-10-3
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: Elasticsearch
tags:
    - Elasticsearh
    - ELK
---

# 设置JVM选项
heap size配置原则:
1. 不超过物理内存的50%

2. 不超过JVM用于压缩对象的指针（compressed oops）的阈值以上; 确切的阈值有所不同，但接近32 GB。日志中查找如下行验证是否超过:
```
heap size [1.9gb], compressed ordinary object pointers [true]
```
3. 不超过zero-based compressed oops的阈值(大多数系统26G是安全的)
查看是否超过:
jvm.options开启(新版-Xlog:gc+heap+coops=info)
```
-XX:+UnlockDiagnosticVMOptions 
-XX:+PrintCompressedOopsMode
```
查看日志
```
heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops     # 不超过(JDK 7)
Heap address: 0x0000000080000000, size: 27648 MB, Compressed Oops mode: 32-bit  # 不超过(JDK 8)
heap address: 0x0000000118400000, size: 28672 MB, Compressed Oops with base: 0x00000001183ff000 # 超过(JDK 7)


$ JAVA_HOME=`/usr/libexec/java_home -v 1.8` java -Xmx32766m -XX:+PrintFlagsFinal 2> /dev/null | grep UseCompressedOops
# 如果启用会提示如下信息
bool UseCompressedOops   := true
```

设置方法一:
通过设置环境变量 ES_JAVA_OPTS
1.注释掉jvm.options中的Xms and Xmx        
2. 
```
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch 
ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch 
```
3. ./bin/elasticsearch

设置方法二:
jvm.options配置文件中设置Xmx/Xms。
```
-Xmx2g      # 独立于版本
8:-Xmx2g    # =v8
8-:-Xmx2g   # >= v8
8-9:-Xmx2g  # v8到v9版本之间
```


# data和logs配置
path.data (可定义多个)和path.logs
```
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch

path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```
# 集群名称和网络
cluster.name: logging-prod    
node.name: prod-data-2     # 默认主机名    
network.host: 192.168.1.10  # 默认loopback addresses    

# 集群发现设置
discovery.seed_hosts设置,若discovery.seed_providers则添加    
包含集群中所有符合主机要求的节点(master-eligible)的地址    
```
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
```
discovery.seed_providers: file # 基于文件$ES_PATH_CONF/unicast_hosts.txt(可动态设置,无需重启es)    
vim $ES_PATH_CONF/unicast_hosts.txt 每行一个host    
```
10.10.10.5
10.10.10.6:9305
10.10.10.5:10005
# an IPv6 address
[2001:0db8:85a3:0000:0000:8a2e:0370:7334]:9301
```
cluster.initial_master_nodes设置    
第一次启动一个全新的集群时必须配置    
```
cluster.initial_master_nodes: 
   - master-node-a
   - master-node-b
   - master-node-c
```

# JVM heap dump path设置
作用是当JVM需要在内存不足异常时执行堆转储(heap dump)    
默认在    
1./var/lib/elasticsearch(包安装)    
2.ES根目录下的data(解压安装)  
```  
jvm.options中设置    
-XX:HeapDumpPath=data 
```
# 临时目录
默认ES会使用一些临时文件(在启动时生成)默认在/tmp目录下    
1.包安装默认排除在/tmp定期删除(不必自定义)    
2.解压安装则要设置$ES_TMPDIR环境变量指向临时目录,该目录只有运行ES服务的用户可访问     

# 系统配置
1. .zip or .tar.gz 
ulimit(临时设置)   
/etc/security/limits.conf(永久生效)    

2.包安装
```
RPM      /etc/sysconfig/elasticsearch
Debian  /etc/default/elasticsearch
```
使用systemd管理的系统
```
sudo systemctl edit elasticsearch
sudo systemctl daemon-reload
```

# 禁用swapping
禁用交换有三种方法，首选的选项是完全禁用交换，如果这不是一个选项，是否选择最小化的swappiness还是内存锁定取决于你的环境。
方法一:
```
sudo swapoff -a
```
要永久禁用它，你需要编辑/etc/fstab文件，并注释掉任何包含单词swap的行。

方法二:
配置 swappiness     
Linux系统上的另一个可用选项是确保sysctl值vm.swappiness设置为1，这减少了内核交换的趋势，在正常情况下不应该引起交换，同时仍然允许整个系统在紧急情况下交换。      
修改swappiness的值      
1）临时设置（重启后失效）

```
# echo 1 > /proc/sys/vm/swappiness
# sysctl -a | grep vm.swappiness
# vm.swappiness = 1
```
 
注：必须以root用户登录


可选方法如下
```
# sysctl -w vm.swappiness=1
vm.swappiness = 1
# cat /proc/sys/vm/swappiness
1
```

2）永久设置
在/etc/sysctl.conf中编辑，增加如下参数（如果存在的话）
```
vm.swappiness = 1
```

方法三:
启用 bootstrap.memory_lock     
尝试将进程地址空间锁定到RAM中，以防止任何Elasticsearch内存被交换出去，这可以通过向config/elasticsearch.yml文件中添加这一行来实现：
```
bootstrap.memory_lock: true
```
请注意:
> mlockall might cause the JVM or shell session to exit if it tries to allocate more memory than is available!

检查memory_lock配置是否成功    
GET _nodes?filter_path=**.mlockall    
如果你看到mlockall为false，那么这意味着mlockall请求失败了    

在Linux/Unix系统上，最可能的原因是运行Elasticsearch的用户没有锁内存的权限，这可以被授予如下：
在启动Elasticsearch之前作为root身份设置    
```
ulimit -l unlimited
```
或在
```
/etc/security/limit.conf中将memlock设置为unlimited。
```
mlockall失败的另一个可能原因是临时目录（通常是/tmp）与noexec选项一起挂载，这可以通过使用ES_JAVA_OPTS环境变量指定一个新的临时目录来解决：
```
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
./bin/elasticsearch
```

# 文件描述符File Descriptorsedit
1.包安装无需设置,默认65535      

2..zip or tar.gz在启动Elasticsearch之前作为root身份设置ulimit -n 65536，或在/etc/security/limits.conf中设置nofile为65536。
检查设置是否生效:
```
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```

# 最大文件大小Max file size
```
ullimit -f unlimited
```
或者在/etc/security/limit.conf中将fsize设置为4096来实现。

# Maximum map count 
```
sysctl -w vm.max_map_count=262144
sysctl -p 
```
要永久设置此值，请更新在/etc/sysctl.conf中的vm.max_map_count设置
> rpm包安装无需设置

# virtual memory
```
ulimit -v unlimited
```
或者在/etc/security/limit.conf中将as设置为unlimited来实现。 - address space limit (KB)

# 线程数
```
ulimit -u 4096
```
或者在/etc/security/limit.conf中将nproc设置为4096来实现。
