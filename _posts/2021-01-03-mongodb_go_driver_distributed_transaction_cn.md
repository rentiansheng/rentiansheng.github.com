---
layout: post
title: Use mongodb driver to solve distributed transactions
category: [项目使用]
tags: [项目使用]
---

[中文](mongodb_go_driver_distributed_transaction_cn.html)

## Use mongodb driver to solve distributed transactions


#### Solving distributed transaction scenarios

This project can only solve distributed transactions in multiple tables in the same database in the same replica-set in mongodb.



#### Main principles

The mongodb server does not strictly bind the transaction in which the transaction is in, and the mongodb driver can initiate transactions to achieve this.
Changing this solution will extend the mongodb golang driver code,
It can be used for verification in mongodb driver 1.1.2 release and mongodb 4.0.x-4.2.x replica-set mode

###### mongodb golang driver code extension content


[The specific code is at https://github.com/rentiansheng/mongodb_driver_transaction/commit/314f4f3263953ea9e6588993975c44cb894e9831](https://github.com/rentiansheng/mongodb_driver_transcation/commit/314f4f3263953ea9e6588993975c44cb894e9831)


Extension code:
```
 mongo/session_exposer.go
 x/mongo/driver/session/session_ext.go
```
The extension code is mainly the underlying logic, used to activate the transaction and bind the transaction. Does not contain business logic



Related test cases:

```
transcation/transaction_test.go

```

Upper-level use logic:

```
transcation/transaction.go
```

Encapsulate business logic, realize business-level management interfaces such as opening, committing, and rolling back transactions, and provide an operation interface for the transaction uuid and the cursor id record operation of the statement execution within the transaction.
In actual use, the transaction uuid and in-service statement execution cursor id need to be stored centrally, and all service instances need to be available and used. You can use redis and mysql as central storage

Now that the real transaction id of mongodb is directly exposed, there may be security risks in passing between different service nodes.

###### Specific implementation

1. Activate a transaction, generate a transaction uuid (mongodb driver provides the generation method), transaction/transaction.go: StartTransaction
2. Activate a session through the uuid of the transaction and join a transaction of the mongodb server, transaction/transaction.go: ReloadSession
3. Bind the transaction to the session, get the SessionContext, mongo/session_exposer.go:TxnContextWithSession
4. Perform curl operations





#### Solution discovery and implementation

participants:

[rentiansheng](https://github.com/rentiansheng)
[wusendong](https://github.com/wusendong)
[breezelxp](https://github.com/breezelxp)

Use items:


[Blueking Configuration Platform](https://github.com/Tencent/bk-cmdb)


#### Drainage
[bson decode register](https://github.com/rentiansheng/bson-register)

Bson decoding interface priority uses the actual type as the final object, such as slice, map
In the mongodb golang official driver, when bson uses interface at the top level, it puts slice and map in an object called primitive.D (golang []interface type).