---
layout:     post
title:      Golang学习笔记八
subtitle:   面向"对象"-Go自定义结构不能导入包?(GOPATH解密)
date:       2019-01-19
author:     CDz
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Golang
---


4369   #erlang发现口

5672   #client端通信口

15672  #管理界面ui端口

25672  #server间内部通信口


rbbitmq官网

https://www.rabbitmq.com/

下载地址
https://www.rabbitmq.com/download.html

下载:RHEL/CentOS 7.x 

由于rabbitmq是erlang编写的,所以需要先安装erlang.

```
rpm -ivh erlang-21.0.9-1.el7.centos.x86_64.rpm
rpm -ivh   socat-1.7.3.2-2.el7.x86_64.rpm
rpm -ivh  rabbitmq-server-3.7.7-1.el7.noarch.rpm
```

离线启动
rabbitmq-server -detached 

关闭:
rabbitmqctl stop

rabbitmq常用命令:
rabbitmqctl add_user admin 123456
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p "/" admin "." "." ".*"

//开启管理页面插件
rabbitmq-plugins enable rabbitmq_management