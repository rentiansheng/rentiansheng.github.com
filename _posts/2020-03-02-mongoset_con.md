---
layout: post
title:  docker mongodb 集群模式Golang连接不上的问题
category: [tcp, network]
tags: [golang]
---


#### 场景

```sh
server selection error: server selection timeout
current topology: Type: ReplicaSetNoPrimary
Servers:
Addr: mongosecond2:27017, Type: Unknown, State: Connected, Average RTT: 0, Last error: connection(mongosecond2:27017[-2108]) connection is closed
Addr: mongosecond3:27017, Type: Unknown, State: Connected, Average RTT: 0, Last error: connection(mongosecond3:27017[-2110]) connection is closed
Addr: mongoprimary:27017, Type: Unknown, State: Connected, Average RTT: 0, Last error: connection(mongoprimary:27017[-2109]) connection is closed
Addr: mongosecond1:27017, Type: Unknown, State: Connected, Average RTT: 0, Last error: connection(mongosecond1:27017[-2107]) connection is closed

```

在终端使用使用mongo client 连接操作没有问题。 

```
rs0:PRIMARY> db.test.insert({"aa":"111"});
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.test.find();
{ "_id" : ObjectId("5e5cb62d5a5225dfab47ed44"), "aa" : "111" }
rs0:PRIMARY>
```


#### 分析问题过程

背景： 

我的项目在宿主机上面运行，数据库mongodb在docker中进行。 mongodb共有四个容器提供服务， 其中一个台是协调者。
项目中mongodb 的配置都是127.0.0.1地址和mongodb 容器在宿主机上暴露的端口

eg:
```
127.0.0.1:27017
127.0.0.1:27018
127.0.0.1:27019
127.0.0.1:27020
```

###### 可能出现的问题一

项目中配置的是IP:PORT，但是错误提示中出现是hostname:port.

hostname 是容器的hostname， 在外侧肯定是无法转换成IP的。 
配置宿主的hosts

###### 可能出现的问题二


我的容器是用docker-compose build 出来. 我在其中使用了
expose和ports暴露容器的端口, 我本意是expose的端口用来mognodb 集群使用，
ports的端口给服务使用。 但是具体使用的时候，提示出现了expose的端口。 







