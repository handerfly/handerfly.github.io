---
layout:     post
title:      Elasticsearch优化
date:       2020-02-20
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: ELK
tags:
    - ELK
    - Elasticsearch
---

更新Cluster API
PUT /_cluster/settings

# 集群基本的分片分配
cluster.routing.allocation.enable  默认all(允许所有分片分配),可选值primaries,new_primaries,none(禁止所有分片)
cluster.routing.allocation.node_concurrent_incoming_recoveries 默认2,同时分配到该节点的recoveries 分片数
cluster.routing.allocation.node_concurrent_outgoing_recoveries 默认2
cluster.routing.allocation.node_concurrent_recoveries  # 同时设置以上两个值
cluster.routing.allocation.node_initial_primaries_recoveries  默认4

cluster.routing.rebalance.enable   默认all(允许所有分片reblanncing),可选值primaries,replicas,none(禁止所有分片)

#设置awareness
#数据节点上面设置rack_one rack_two,rack_two down则会把分配都分配给rack_one  
node.attr.rack_id: rack_one
#在所有master节点 master-eligible 配置 elasticsearch.yml 
cluster.routing.allocation.awareness.attributes: rack_id

#node.attr.zone 设置为zone1，则只会分配主分片，知道zone2的节点可用
cluster.routing.allocation.awareness.attributes: zone
cluster.routing.allocation.awareness.force.zone.values: zone1,zone2


# 排除节点
include(or)
require(and)
exclude(not)
```
PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.*"
  }
}
```
# 基于磁盘的分片分配
```
cluster.routing.allocation.disk.threshold_enabled      默认true, 启用磁盘空间阈值检查
cluster.routing.allocation.disk.watermark.low          默认85%, 分片不会分配(除新建的索引的主分片不受影响)
cluster.routing.allocation.disk.watermark.high         默认90%, 达到该值,ES会做relocate,影响所有分片
cluster.routing.allocation.disk.watermark.flood_stage  默认95%, index.blocks.read_only_allow_delete,当磁盘使用率低于95%,自动释放index.blocks
cluster.info.update.interval    默认30s设置多久检查一次磁盘
```
设置disk watermark 可以具体值或百分比100gb,85%
```
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "85%",
    "cluster.routing.allocation.disk.watermark.high": "90%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "95%",
    "cluster.info.update.interval": "1m"
  }
}
```



# 设置recovery
```
PUT /_cluster/settings
{
    "persistent" : {
        "indices.recovery.max_bytes_per_sec" : "50mb"
    }
}
#reset
PUT /_cluster/settings
{
    "transient" : {
        "indices.recovery.max_bytes_per_sec" : null
    }
}
```

# 设置索引read-only index block
```
#null还原默认
PUT /twitter/_settings
{
  "index.blocks.read_only_allow_delete": null
}
#false表示可以允许删除
PUT _all/_settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```

# 索引基本分配
```
node.attr.size: medium

PUT test/_settings
{
  "index.routing.allocation.include.size": "big,medium"
}

PUT test/_settings
{
  "index.routing.allocation.include.size": "big",
  "index.routing.allocation.include.rack": "rack1"
}

PUT test/_settings
{
  "index.routing.allocation.include._ip": "192.168.2.*"
}
```
# 限制node索引的分片数
index.routing.allocation.total_shards_per_node
# 限制node的总分片数
cluster.routing.allocation.total_shards_per_node



# 开启慢查询日志
```
curl -XPUT http://localhost:$ES_PORT/_cluster/settings -H ‘Content-Type: application/json’ -d’
{
"transient" : {
"logger.index.search.slowlog" : "DEBUG",
"logger.index.indexing.slowlog" : "DEBUG"
}
}'
```

#所有慢速日志记录都在索引级别启用，设置为0s以分析实例并收集正在发送的所有查询，并设置为-1以关闭慢速日志
```
PUT /twitter/_settings
{
    "index.search.slowlog.threshold.query.warn": "10s",
    "index.search.slowlog.threshold.query.info": "5s",
    "index.search.slowlog.threshold.query.debug": "2s",
    "index.search.slowlog.threshold.query.trace": "500ms",
    "index.search.slowlog.threshold.fetch.warn": "1s",
    "index.search.slowlog.threshold.fetch.info": "800ms",
    "index.search.slowlog.threshold.fetch.debug": "500ms",
    "index.search.slowlog.threshold.fetch.trace": "200ms",
    "index.search.slowlog.level": "info"
}

curl -XPUT http://localhost:$ES_PORT/*/_settings?pretty -H 'Content-Type: application/json' -d 
'{"index.search.slowlog.threshold.query.debug": "-1",
 "index.search.slowlog.threshold.fetch.debug": "-1",}'
```

[参考](https://www.linuxprobe.com/screen-example.html)