---
layout:     post
title:      模板
date:       2019-07-22
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: Elasticsearch
tags:
    - Elasticsearch
---

# elasticsearch-head使用说明
ealsticsearch只是后端提供各种api，那么怎么直观的使用它呢？elasticsearch-head将是一款专门针对于elasticsearch的客户端工具

elasticsearch-head配置包，下载地址：[elasticsearch-head下载地址](https://github.com/mobz/elasticsearch-head)

elasticsearch-head是一个基于node.js的前端工程，启动elasticsearch-head的步骤如下（这里针对的是elasticsearch 5.x以上的版本）：

1、进入elasticsearch-head的文件夹，如：D:\xwj_github\elasticsearch-head

2、执行 npm install

3、执行 npm run start

在浏览器访问http://localhost:9100

绿色，最健康的状态，代表所有的分片包括备份都可用
黄色，基本的分片可用，但是备份不可用（也可能是没有备份）
红色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
灰色，未连接到elasticsearch服务

![getopts](https://raw.githubusercontent.com/handerfly/handerfly.github.io/master/img/getopt.png)  

[elasticsearch-head下载地址](https://github.com/mobz/elasticsearch-head)
[参考](https://www.cnblogs.com/xuwenjin/p/8792919.html)