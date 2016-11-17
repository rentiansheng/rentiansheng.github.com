---
layout: post
title: 服务治理
category: [服务治理]
tags: [服务治理]
---

# 服务治理

## 为什么要做服务治理

一个系统在开发之初，业务单一，系统规比较小。 随着业务发展，系统会慢慢的变的臃肿。

会出现下列问题：

1. 代码维护和部署变得困难
2. 水平扩展困难
3. 功能及技术迭代困难
（注：技术迭代指的是替换使用的语言、第三方工具等）

为了解决上述的问题，我们会开始分拆系统，将一个臃肿的系统拆分成N个小系统。

拆分会带来下列好处：

1. 独立部署，方便水平扩展
2. 系统隔离
3. 快速的迭代（每个项目可以根据自己的业务场景来决定使用最优的语言和第三方工具）

但是，拆分并没有从根本上解决问题，随着业务发展，我们拆分出来的N个小系统继续添加相关的业务，
随着时间的发展也变的复杂和臃肿。我们又继续拆分。 然后系统又开始从简单变得臃肿，
一遍又一遍做着重复的事情。


我们如何解决这个问题：
在接到开发任务之初，将任务做成一个或者多个服务单元（服务单元:请看下面注释），每个服务单元单独部署和迭代升级。
禁止添加其他无关的功能。每个单元职责单一，单独部署。

注解：

服务单元：尽可能小的集合，对于一个抽象属性的增删改查，杜绝添加其他相关联的功能和业务。

随之带来的问题是：

1. 独立的服务单元越来越多，如何做服务单元的管理
2. 如何做负载均衡
3. 如何做服务信息的修改及快速生效


最开始的解决方案是基于OP和RD配合来做的。
通过DNS,NGINX,CONFIG文件来配合完成任务。

    DNS，NGINX都可以做负载均衡的功能。

    DNS,NGINX,CONFIG都可以用来管理服务单元信息的变更

上述的方法，虽然可以解决问题，但是每次必须修改配置，重新加载配置才会生效。无法做到快速响应。
为了解决这些问题，就要使用更有效的服务治理方案。

## 服务治理

服务治理是围绕服务信息管理进行。为了完成这个任务及结合使用的场景，
我们需要一个高可用、低延迟的数据存储工具。

经过筛选我们最终选择了zookeeper。
我们先来zookeeper官方的定义：

ZooKeeper is a centralized service for maintaining configuration information,
 naming, providing distributed synchronization, and providing group services. 

使用zookeeper原因：

1. 公司已经有zookeeper集群，具备运维能力
2. zookeeper 使用范围比较广，
3. watcher机制及时通知修改，保证信息一直

虽然，我们可以通过zookeeper来解决服务配置的问题，
但是服务的信息并不会自动出现在zookeeper中，需要我们开发一个完整的功能。

服务治理需要的功能：

1. 服务发现

    主动注册和第三方注册，我使用的是被动注册，因为调用方和服务方都是PHP，主动注册实现复杂

2. 服务管理

    服务统计信息、服务信息、授权查看，在第三方注册时候，提供服务注册和修改功能

2. 通信

    异步通信和同步通信，现在只实现同步通信


2. 访问控制

    限制调用方key对服务方访问频次、是否具有访问权限

5. 数据交互协议适配

     调用放不用关注服务方提供服务协议。目前支持http下json、yar-msgpack相互转换



## 如何去做

到目前为止，我找到解决问题的方法和需要实现的功能的。
根据上面的信息，我来看下我们具体是的设计。


![](/img/zkmanagerservice.png)

在设计中我们工分五部分：

1. 调用者

    通过HTTP或YAR访问API Gateway， 将需要调用的服务信息和参数告诉API Gateway，

2. 服务方

    a) 提供服务

    b) 将自己的信息通过管理平台注册到zookeeper中
         
         1. 服务名、状态
         2. 地址信息、机房、权重及状态
         3. 服务下接口列表
             a) 接口名
             b) 接口路径
             c) 输出数据的编码（json或者yar-msgpack）
             d) 状态


3. API Gateway

    a) 转发请求

    b) 服务不存在或者不提供服务状态下快速失败

    c) 对发送的数据做编码（json或者yar-msgpack）

    d) 负载均衡

    e) 将服务和接口的信息缓存到内存中

    f) 通过watcher机制将变更的服务信息缓存到内存中


4. 管理平台

    a) 服务及接口注册

    b) 服务及接口查看

    c) 服务及接口修改

    d) 统计信息查看


4. ZooKeeper

   ZooKeeper在整个服务注册与发现的设计中，最重要是如何来存储服务的注册信息。
   在使用zookeeper我们先来看下zookeeper是如何存储数据的。
   下面是zookeeper官方的介绍:

   The name space provided by ZooKeeper is much like that of a standard file system. 
   A name is a sequence of path elements separated by a slash (/). 
   Every node in ZooKeeper's name space is identified by a path.

![](http://zookeeper.apache.org/doc/r3.1.1/images/zknamespace.jpg)
  
   
   
   zookeeper就像一个树，用每一个"/{name}"表示节点及节点名。每一个节点可以有子节点和内容。


   我们在zookeeper用于实现服务治理的信息有服务信息、地址信息、接口信息。具体格式如下：

   服务信息格式：/service/{服务名}

   地址信息：/address/{服务名}

   API信息：/api/{服务名}/{接口名}

   为什么要将zookeeper的存储结果设置上面的结果， zookeeper在watcher处理给了两个API（golang zk）
   一个是关注子节点变化ChildrenW，一个是关注本身节点化GetW。这样设计让每个节点功能单一。
   


## 出现的问题（如何保证缓存信息和zookeeper一致）

Gateway 为了做到低延迟和高可用性，在AGateway 中缓存服务配置，
既然有缓存那么必须要涉及到缓存与zookeeper配置信息一致， 
就需要用到zookeeper的watcher机制。在zookeeper中内容中修改时，
通过watcher机制通知API Gateway 来更新缓存

具体的使用方法如下：

watcher 正常情况：

![](/img/zkwatcher.png)

watcher 错误情况:

![](/img/zkwatcherror.png)




 


