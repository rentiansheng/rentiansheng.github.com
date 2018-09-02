---
layout: post
title:  共享分布式锁
category: [distributed lock]
tags: [distributed lock]
---

#### 前提

分布式锁是在分布式系统中，我们为了保证分布式系统的效率和数据的正确性，在相同工作的多个节点中不被重复处理而采用的技术的。



#### 使用场景

现在看网上的分布式锁都是现在资源限制。 锁的使用者限定到当前服务使用者。 在分层的web锁无法继承。
在系统设计中我们经常会才分层设计， 会有负责处理逻辑的层，处理数据的存储层。为了保证服务的高可用每一个服务我们都会部署多个实例。 
为了保证数据的完整性，我们需要在逻辑锁住资源， 在数据处理层修改数据。   这里面就涉及到两个问题

1. 锁如何共享？
2. 如何获取锁？



####  实现


###### 分布式锁实现

为了保证分布式锁的正确性，我们可以选择以下方案。

1. redis SETNX 方案
2. redisLock 方案
2. zookeeper 顺序节点


##### 锁的结构设计

注解：由于锁是在分布式事务中事务，其中定义变量与实务相关, 代码是golang

``` 

锁的结构

type Lock struct {
	//  当前锁的标记。 是开启事务得到的事务ID， 事务主ID，不可以为空
	TxnID string `json:"txnID"`

	// 锁资源的子事务ID， 可以为空， 为空将自动生成改项目
	SubTxnID string `json:"subTxnID"`

	// 被锁资源的标记或者名字
	LockName string `json:"lockName"`

	
	// 锁的超时时间
    Expire time.Duration `json:"expire"`

    // 锁的创建时间
	Createtime time.Time `json:"createTime"`
}


锁的返回值， 无法锁住资源时，返回nil （就是空NULL）
type LockResult struct {
	// 锁资源的子事务ID， 
	SubTxnID string `json:"subTxnID"`

	// 获取lock 传入的TxnID事务中是否有子事务拥有锁，
	Locked bool `json:"locked"`

	// 拥有锁的子事务ID， 及时第一lock资源的子事务ID
	LockSubTxnID string `json:"lockSubTxnID"`
}

```

##### 如何获取锁

- 用户需要携带TxnID 和 SubTxnID, TxnID必填， SubTxnID非必填，如果SubTxnID没有填写，则自动生成。 
- 使用原子操作写入原子锁， 如果写入成功，则表示当前TxnID和SubTxnID拥有锁。
- 使用原子操作写入原子锁， 如果写入失败，获取当前锁的内容， 先判断锁中的TxnID是否与传入的TxnID相等， 如何不等于返回无法锁住资源，   
否则TxnID等于TxnID， 继续执行
- 判断SubTxnID是否等于传入的SubTxnID， 等于则返回当前TxnID和SubTxnID拥有锁， 否则返回TxnID拥有锁， SubTxnID不拥有锁




#### 如何共享锁

逻辑层调用存储层的时候通过HTTP header 将TxnID 透传下去














