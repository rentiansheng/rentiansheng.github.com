---
layout: post
title:  read preference in a transaction must be primary
category: [http]
tags: [http]
---


### 背景

项目中mongodb数据库负载比较高，现在想将读请求放到secondary节点中， 但是将ReadPreference设置为SecondaryPreferred， 
事务中的查询语句会出现read preference in a transaction must be primary的问题

### 解决方案

- 修改操作数据的逻辑
- 修改mongodb driver事务内节点直接使用primary 节点操作 



#### 修改操作数据的逻辑

思想： 

mongodb driver 在对mongodb driver 中的 DataBase,Collection的对象操作的时候允许通过Option 来在设定执行的ReadPrefernce的。  

mongodb driver 关于DataBase，Collection 对象操作的定义如下：

```
Database(name string, opts ...*options.DatabaseOptions) *Database
Collection(name string, opts ...*options.CollectionOptions) *Collection
```

具体实现 ：

- 1. 修改连接方法， 设置连接mongodb 使用Option 中的ReadPrefernce值为SecondaryPreferred
- 2. 根据ctx 判断当前是否位事务， 如果是设置  DataBase或Collection的Option中的ReadPrefernce值为SecondaryPreferred 

方案优点：
- 逻辑简单
- 对业务逻辑没有侵入性 

方案确定：
- 可移植性差
- 不支持标准事务




#### 修改mongodb driver 


思想：

ReadPrefernce 配置主要设置mongodb driver 在query 数据的时候选择使用连接的策略。 理论上事务内的查询和写操作是一样，必须要走Primary节点， 所以可以在driver中将事务的查询的方法强制设置位Primary即可。 

代码如下：
[修改代码连接](https://github.com/mongodb/mongo-go-driver/pull/501)

方案优点：
- 对业务逻辑没有侵入性 
- 可移植性
- 支持标准事务
- 逻辑简单

方案确定：
-  非官方修改，兼容性未知