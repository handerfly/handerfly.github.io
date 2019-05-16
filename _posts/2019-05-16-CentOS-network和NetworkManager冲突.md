---
layout:     post
title:      CentOS-network和NetworkManager冲突
date:       2019-05-16
author:     BenderFly
header-img: img/post-bg-coffee.jpeg
catalog: true
categories: 网络
tags:
    - centos7
---

# CentOS - network和NetworkManager冲突
在CentOS系统上，目前有NetworkManager和network两种网络管理工具。
如果两种都配置会引起冲突，而且NetworkManager在网络断开的时候，会清理路由，如果一些自定义的路由，没有加入到NetworkManager的配置文件中，路由就被清理掉，网络连接后需要自定义添加上去。

# 解决冲突


目前在CentOS上的NetworkManager版本比较低，而且比较适合有桌面环境的系统，所以服务器上保留network服务即可，将NetworkManager关闭，并且禁止开机启动。


# systemd管理上
```
systemctl status NetworkManager #查看状态
systemctl stop NetworkManager
systemctl disable NetworkManager

Removed /etc/systemd/system/multi-user.target.wants/NetworkManager.service.
Removed /etc/systemd/system/dbus-org.freedesktop.NetworkManager.service.
Removed /etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service.

systemctl is-enabled NetworkManager #查看是否禁用
```

可以看到关联的几个服务一起被禁用了。如果使用桌面的话，会发现网络管理的图标不见了。


# sysv+upstart管理上
```
service NetworkManager stop
chkconfig NetworkManger off
```
