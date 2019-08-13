---
layout:     post
title:      Elasticsearch 别名
date:       2019-07-10
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: Django
tags:
    - Elasticsearch
    - alias
---

别名解决了哪些问题？
回到顶部
在开发中，随着业务需求的迭代，较老的业务逻辑就要面临更新甚至是重构，而对于es来说，为了适应新的业务逻辑，可能就要对原有的索引做一些修改，比如对某些字段做调整，甚至是重建索引。而做这些操作的时候，可能会对业务造成影响，甚至是停机调整等问题。由此，es提供了索引别名来解决这些问题。
索引别名就像一个快捷方式或是软连接，可以指向一个或多个索引，也可以给任意一个需要索引名的API来使用。别名的应用为程序提供了极大地灵活性，别名允许我们做下面这些操作：

在运行的集群中可以无缝的从一个索引切换到另一个索引。
可以给多个索引分组。
给索引的一个子集创建视图，没错我们可以简单将es中的索引理解为关系型数据库中的视图。
可以与路由搭配使用。
当然，既然是别名，就不能与索引同名，并且别名仅支持查询（暂时可以这么说）操作。因为如果有多个索引指向同一个别名的话，es不知道你要对哪一个具体的索引做操作。

别名的相关操作
回到顶部
准备数据
回到顶部
首先，先准备两个索引：

PUT l1/doc/1
{
  "title":"我想要睡你"
}

PUT l2/doc/1
{
  "title":"你却拿我当兄弟"
}

PUT l3/doc/1
{
  "title":"不过，我不介意"
}
创建别名
回到顶部
我们来为一个索引起一个别名：

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l1",
        "alias": "a1"
      }
    }
  ]
}
上例，我们使用add为索引l1添加一个别名a1。

查看别名
回到顶部
来查看一下刚才添加的别名：

GET l1/_alias
# result
{
  "l1" : {
    "aliases" : {
      "a1" : { }
    }
  }
}
删除别名
回到顶部
删除别名使用remove：

POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "l1",
        "alias": "a1"
      }
    }
  ]
}
这样就删除了指向索引l1的别名a1，再查l1的别名，你会发现aliases为空。注意，重复删除别名会报aliases_not_found_exception错误。

重命名别名
回到顶部
重命名别名是一个简单的remove操作，然后执行add操作，无需担心短时间内别名不指向索引，因为这个操作原子性的：

POST /_aliases
{
  "actions": [
    {"remove": {"index": "l1", "alias": "a1"}},
    {"add": {"index": "l2", "alias": "a1"}}
  ]
}
上例，首先索引l1又要有别名a1（别名都没有，删除等着报错呢）。这个原子性的操作，首先删除l1的别名a1，然后添加索引l2的别名a1。这个操作很有爱啊，因为如果要淘汰掉旧的索引l1并将所有数据迁移到了新索引l2上后，这时，我们如果一直使用a1别名处理请求，经过这个操作，在用户毫无感知的情况下就完成了索引的变更。

为多个索引指向同样的别名
回到顶部
为多个索引指向同样的别名只需要几个add操作就OK了：

POST /_aliases
{
  "actions": [
    {"add": {"index": "l1", "alias": "a1"}},
    {"add": {"index": "l2", "alias": "a1"}},
    {"add": {"index": "l3", "alias": "a1"}}
  ]
}
上例，我们将l1、l2、l3三个索引都指向别名a1。

使用indices数组语法在一个操作中为多个索引指向同一个别名
回到顶部
可以使用indices数组语法在一个操作中为多个索引指向同一个别名，也是上面示例的另一种写法：

POST /_aliases
{
  "actions": [
    {"add": {"indices": ["l1", "l2", "l3"], "alias": "a2"}}
  ]
}
当然，这个套路同样适用于在一个操作中为一个索引指向多个别名：

POST /_aliases
{
  "actions": [
    {"add": {"index": "l1", "aliases": ["a1", "a2", "a3"]}}
  ]
}
对于上面的示例，也可以使用glob pattern将别名关联到拥有公共名称的多个索引：

POST /_aliases
{
  "actions": [
    {"add": {"index": "l*", "alias": "f1"}}
  ]
}
上例指将所有以l开头的索引都指向别名f1。这个操作非常的有用，比如我们处理日志的逻辑是以日期为索引名，每天创建一个索引，那么我们如果想以月为单位分析日志，就可以采用这种方式，比如将2月份每一天的索引都指向一个别名。便于操作。

别名交换
回到顶部
在一个操作中使用别名交换：

POST /_aliases
{
  "actions": [
    {"add": {"index": "l1", "alias": "a1"}},
    {"remove_index":{"index":"a1"}}
  ]
}
上例的例子的意思是，开始，我们错误的添加了索引a1，而是l1才是我们想要添加的索引，所以这里我们可以先为索引l1创建一个别名a1，然后再把原来的索引a1干掉。remove_index就像删除索引一样（是的，索引a1就这样被干掉了）。

过滤器别名
回到顶部
带有过滤器的别名提供了创建相同索引的不同视图的简单方法，过滤器可以使用查询DSL定义，并应用与所有的搜索、计数、按查询删除以及诸如此类的操作。
要创建一个带过滤器的别名，首先要确保映射字段已存在与mapping中：

PUT l4
{
  "mappings": {
    "doc":{
      "properties":{
        "year":{
          "type":"integer"
        },
        "method":{
          "type":"keyword"
        }
      }
    }
  }
}

PUT l4/doc/1
{
  "year":2019,
  "method":"GET"
}
PUT l4/doc/2
{
  "year":2018,
  "method":"POST"
}
PUT l4/doc/3
{
  "year":2019,
  "method":"POST"
}
我们来看过滤器别名是如何创建的：

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l4",
        "alias": "a4",
        "filter": {"term": {
          "year": 2019
        }}
      }
    }
  ]
}
上例，我们为索引l4以year字段建立了别名a4。我们来看一下它与普通的别名有什么区别：

{
  "l4" : {
    "aliases" : {
      "a4" : {
        "filter" : {
          "term" : {
            "year" : 2019
          }
        }
      }
    }
  }
}
如果你够细心的话，你会发现这个带过滤器的别名相较于之前的别名多了过滤条件，普通的别名aliases的字典是空的。
那么有啥用呢？我们如果以索引l4查询，一切没啥变化：

GET l4/doc/_search
上例查询返回该索引内的所有文档。
而如果以别名a4查询则仅返回过滤器过滤后的结果：

GET a4/doc/_search
上例仅返回year为2019的文档。

与路由连用
回到顶部
除了单独使用别名，还可以将别名与路由值关联，以避免不必要的分片操作。

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l1",
        "alias": "a1",
        "routing": "1"
      }
    }
  ]
}
上例为索引l1创建了一个别名a1，在创建完a1后，所有与此别名的操作都将自动修改使用routing值进行路由。
除此之外，还可以为搜索和索引操作指定不同的路由值：

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l4",
        "alias": "a4",
        "search_routing": "1,2",
        "index_routing": "1"
      }
    }
  ]
}
如上例所示，搜索路由（search_routing）可能包含多个（以英文逗号分隔）值，但索引路由（index_routing）就只能包含一个值。
如果使用路由别名的搜索也有路由参数，则使用在该参数中指定的搜索别名路由和路由的交集：

GET /a4/_search?q=year:2019&routing=2,3
上例中，搜索操作中有路由参数2、3，而搜索路由设置的是1、2，所以去交集2。

写索引
回到顶部
在之前的介绍中，我们说过，我们仅能使用别名进行查询操作，不能做写入操作，因为如果一个别名关联多个索引的话，es不知道你要将文档写入到哪个索引中，而现在，我们就来解决这个问题，我们要告诉es，要将文档写入到那个索引中。

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l1",
        "alias": "a1",
        "is_write_index": true
      }
    },
    {
      "add": {
        "index": "l2",
        "alias": "a1"
      }
    }
  ]
}
注意，上述的l1和l2索引必须存在。
在上例中，is_write_index:true表示当通过别名a1做写入操作的时候，将文档写入到索引l1中。

PUT a1/doc/2
{
  "title": "hi gay"
}
上例中，PUT a1/doc/2相当于向l1/doc/2写入。

GET l1/doc/_search
GET l1/doc/2
那么如果有些文档的写入要从l1切换到l2该怎么办呢？要交换哪个索引是别名的写入操作，可以利用别名API进行原子交换，交换不依赖于操作的顺序。

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l1",
        "alias": "a1",
        "is_write_index": false
      }
    },
    {
      "add": {
        "index": "l2",
        "alias": "a1",
        "is_write_index": true
      }
    }
  ]
}
上例中，我们首先取消l1的写索引权限，然后将该权限交给l2。这样，再写入的话，就是向l2索引了。

PUT a1/doc/3
{
  "title": "hi man"
}
GET l2/doc/3
注意，如果一个别名仅有一个索引指向它时，那么该索引是有写权限的：

PUT l5/doc/1
{
  "title":"肉肉的猪仔真可爱"
}

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l5",
        "alias": "a5"
      }
    }
  ]
}
PUT a5/doc/2
{
  "title":"可爱可爱真可爱"
}

GET l5/doc/_search
如上例所示，此时的别名a5仅有索引l5指向它。所以可以通过该别名做写入操作。
但是，当又有新的索引l4指向该别名时，如果没有显示的设置is_write_index:true，此时就不能通过该别名做写索引操作了。

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "l4",
        "alias": "a5"
      }
    }
  ]
}


PUT a5/doc/3     
{
  "title":"可爱可爱真可爱"
}
上例中，当PUT a5/doc/3时会报错illegal_argument_exception，因为此时没有为哪个索引指定is_write_index:true，es就不知道往哪个索引做写索引操作了。

添加单个别名
回到顶部
除了之前添加别名的方法，我们还可以使用端点添加别名：

PUT /{index}/_alias/{name}
PUT /{index}/_alias/{name}?routing=user1
index，要为哪个索引添加别名。
name，别名的名称。
routing，可以与别名关联的路由。
来个示例，基于时间的别名：

PUT /logs_201905/_alias/2019
再来一个示例，为用户添加别名：

PUT user
{
  "mappings": {
    "doc":{
      "properties":{
        "user_id":{
          "type":"integer"
        }
      }
    }
  }
}
PUT /user/_alias/user_1
{
  "routing": "1",
  "filter": {
    "term": {
      "user_id": "1"
    }
  }
}

PUT /user/doc/1?routing=1
{
  "user_id": "1"
}
GET /user/doc/_search
GET /user/doc/1
索引期间指定别名
回到顶部
我们除了单独的为一个或多个索引指定别名，也可以在索引创建期间指定别名：

PUT l6
{
  "mappings": {
    "doc":{
      "properties":{
        "year":{
          "type":"integer"
        }
      }
    }
  },
  "aliases": {
    "current_day": {},
    "2019":{
      "filter": {
        "term": {
          "year": 2019
        }
      }
    }
  }
}
PUT l6/doc/1
{
  "year": 2018
}
PUT l6/doc/2
{
  "year": 2019
}
GET 2019/_search
删除别名
回到顶部
通过使用DELETE删除别名：

DELETE /l1/_alias/a1
DELETE /l2/_aliases/a*
检索现有别名
回到顶部
使用GET索引别名API检索别名：

GET /{index}/_alias/{alias}
index，获取别名的索引名称。通过通配符支持部分名称，也可以使用逗号分隔多个索引名称。还可以使用索引的别名。
alias，要在响应中返回的别名的名称。与index选项一样，此选项支持通配符，并且选项指定由逗号分隔的多个别名。
ignore_unavailable，如果指定的索引名称不存在该怎么办。如果设置为true则那些索引将被忽略。
来一些示例：
查询索引指定别名。

GET l1/_alias/a*    # 查询索引l1指向以a开头的所有别名
GET l1/_alias/*     # 查询索引l1所有的别名
查询所有别名是a1或2019的索引：

GET /_alias/a1
GET /_alias/2019
查询所有别名是以20开头的索引：

GET /_alias/20*
HEAD
回到顶部
通过HEAD检测某个别名是否存在：

HEAD /_alias/2019
HEAD /_alias/20*
HEAD /l2/_alias/*
上例中，HEAD以状态码的形式返回是别名是否存在，200 - OK是存在，404 - Not Found是不存在。

![getopts](https://raw.githubusercontent.com/handerfly/handerfly.github.io/master/img/getopt.png)  

[参考](https://www.cnblogs.com/Neeo/articles/10897280.html)