---
layout: post
title: 服务治理之zookeeper使用中遇到的问题及设计
category: [服务治理]
tags: [服务治理]
---



#  如何规划需要管理配置

根据需求将所有的信息分成四类

    1. 分类

        /classify下每个节点是一个分类名，内容是分类描述，分类属于辅助信息，没有太大意义。

        eg: /classify/c1  分类名c1, /classify/c1节点内容为c1的描述

    2. 服务信息

        /service下每个节点都是一个服务

        eg: /service/srv1  服务名为srv1，/service/c1节点内容服务srv1具体内容

    3. 服务地址

        /address 下每个节点都是一个服务

        eg: /service/srv1  服务名为srv1，/service/c1节点内容服务srv1具体内容

    4. 服务下的API

        /api下每一个节点是一个服务，服务下节点是API的命名的节点

        eg: /api/srv1/api1   /api/srv1下的子节点是srv1服务下的所有API，/api/srv1/api1  服务srv1下api1的内容



# 问题

    1. zk reconnect后授权问题

    2. zk session 过期

    3. zk reconnect 数据同步的问题

    2. watcher 消失后修改的问题

# 功能设计

|模块|功能||
|--|--|--|
|zk|1.管理节点</br>2.内容节点|
|watcher|1.关注子节点变化</br>2.关注本身变化|
|queue|1.处理错误</br>2.重新初始化监控</br>3.初始化缓存</br>4.通知cache变更|
|cache|1.根据类型更新缓存|所有缓存相关的操作|

# 好处

    1. 各模块职责单一

    2. 统一的错误处理

    3. 收敛缓存处理逻辑

    4. 方便扩展

# 解决问题方法

1. zk reconnect后授权问题 

2. zk session 过期

3. zk reconnect 数据同步的问题

    
    问题1、2、3 在queue模块中添加统一的错误处理逻辑

    1. 添加授权信息

    2. 关闭原有的watcher

    3. 初始化watcher进程


  4. watcher 消失后修改的问题

      通过设计queue和cache模块，分开实现，延时获取数据的。

      具体流程，见下图

![](http://7xi8r0.com1.z0.glb.clouddn.com/watcher_note.png)