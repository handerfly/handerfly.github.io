---
layout:     post
title:      windows下MySQL 5.7+ 解压缩版安装配置方法
subtitle:   Bulid Settings -> Documentation Comments -> NO
date:       2019-04-12
author:     BenderFly
header-img: img/post_bg_debug.png
catalog: true
categories: Mysql
tags:
    - Mysql

---

#
1.去官网下载.zip格式的MySQL Server的压缩包，根据需要选择x86或x64版。
2.解压缩至你想要的位置。

3.复制解压目录下my-dafault.ini至bin目录下，重命名为my.ini。并添加以下内容。没有data目录不要紧，下一步处理这个事情。
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:\mysql-5.7.12-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\mysql-5.7.12-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```
4.从MySQL 5.7开始，Oracle公司开始致力于破坏MySQL的易用性，迫使用户使用Oracle数据库。开个玩笑啦。可是没有data文件夹使得网上很多配置方法无效，如果不进行初始化的话，mysql服务是无法启动的。下面是初始化的方法：

　　（1）以管理员身份运行cmd，并cd到mysql中的bin目录下，执行命令：mysqld --initialize --user=mysql --console

　　（2）该命令会创建data目录与数据库，生成root用户和临时密码，如下图，我们需要记住这个命令以便于登录。


> 这种错误是由于未安装 vcredist 引起的（而且版本是 2013版，32位），百度搜索：“vcredist 2013 x86”   

5.配置环境变量，否则你每次都要cd到bin目录下才能使用mysql。右键此电脑（计算机）-属性-高级系统设置-高级-环境变量，在系统变量中的PATH中加入你的bin目录，如：C:\mysql-5.7.12-winx64\bin，点确定！一般不需要重启，如果需要，当我没说。

6.安装MySQL服务，以管理员身份运行cmd，并输入mysqld install MySQL --defaults-file="C:\mysql-5.7.12-winx64\bin\my.ini"，其中的路径为你正式的ini文件。

7.运行cmd，输入net start mysql启动MySQL服务，再输入mysql -u root -p，然后输入临时密码。修改密码：set password = password('新密码');或者:ALTER USER USER() IDENTIFIED BY 'newpasswd';，然后回车就可以了，注意分号不能省略。

就是这样。
