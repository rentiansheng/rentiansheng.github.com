---
layout: post
title: 使用mgo.v2 出现 i/o timeout session 问题
category: [mgo.v2]
tags: [mgo.v2]
---



###  背景

使用mgo.v2 操作数据库，一直报 i/o timeout session 问题出错如下:

```code
read tcp 127.0.0.1:42577->127.0.0.1:27017: i/o timeout
```

### 原因

再上层使用的时候， 先connect mongodb server , 然后 cache connect session. 

再使用mgo.v2 去操作mongodb 的时候，我门都设置操作执行的超时时间， 再超时后mgo.v2 回关闭这个连接，
mgo.v2 只是关闭这个连接，后需通过这个session 或者是 Clone出来的session都是回出现i/o timeout，
mongodb server 日志会有 ` Error sending response to client: SocketException: Broken pipe.` 的错误

参考代码：

```golang
1. 获取网络数据失败或者超时错误，  readLoop(), https://github.com/go-mgo/mgo/blob/v2/socket.go#L551
2. 处理出错连接，  kill(err error, abend bool) https://github.com/go-mgo/mgo/blob/v2/socket.go#L319
3. 再连接上执行语句直接报错的位置， Query(ops ...interface{}) ， https://github.com/go-mgo/mgo/blob/v2/socket.go#L491
```

### 解决方案

```go 

1. 获取session 的时候, 不使用原有session上Clone 使用Copy 方法获取
2. 获取session 的时候， 通过原有session 上clone 出来的 Session， 需要是用Refresh 方法，

```

以上的方法存在性能问题， 每次获取session 都需要和mongodb 进行一次认证服务。 （之前遇到的问题，当时的数据忘记留存了，QPS增高后，严重性能下降）


专用解决方案：
为了快速解决这个问题，并且少修改业务代码和mgo.v2 的代码。
为了保证性能， 对mgo.v2  session 进行扩展， 可以获取session 对应的连接是否已经关闭， 如果关闭再Refresh session。 

在mgo.v2/session.go 新加方法
```go
// 检查session 中连接是否已经dead
func (s *Session) IsDead() bool {
	if s.masterSocket != nil {
		s.masterSocket.Lock()
		if s.masterSocket.dead != nil {
			s.masterSocket.Unlock()
			return true
		}
		s.masterSocket.Unlock()
	}
	if s.slaveSocket != nil {
		s.slaveSocket.Lock()
		if s.slaveSocket.dead != nil {
			s.slaveSocket.Unlock()
			return true
		}
		s.slaveSocket.Unlock()
	}

	return false
}
```

具体使用方法：

```go 
//  如果发现session 中 conn 对象已经dead， 刷新下连接
func (c *MyMongo) refreshDBSess(sess *mgo.Session) *mgo.Session {
	if sess.IsDead() {
		sess.Refresh()
	}
	return sess	
}	

```


##### 其他问题

1. clone session 扩散I/O timeout 和设置 readPreference 也有关， Eventual 不会出现问题（没太关注这个逻辑，按照mgo.v2 通用处理的逻辑，这个是有性能问题）
2. clone session 在mongodb server 重新选择也会有问题。 也可以按照上面的方案解决 
3. 没有看到mgo.v2 连接池的作用





