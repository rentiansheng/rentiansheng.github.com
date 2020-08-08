---
layout: post
title:  调用第三方接口返回过慢的分析过程
category: [http]
tags: [http]
---


### 背景

最近在对第三方系统过程中经常某一个请求耗时很长（请求到返回超过2m）。通过验证发现请求流程中网络传输和业务逻辑处理的耗时并不长。那么时间都耗费到什么时间了？ 
 

### 查找问题的流程

按照个人经验HTTP 的请求流程的耗时是在网络传输和业务逻辑的处理中，现在根据这个流程来确定问题。

#### 确定是否网络问题

1. 通过长时间的ping 发现网络质量比较稳定，时间都在3ms 之内。
2. 通过用strace -p pid 发下connect 并没有失败，一直在等待数据数据返回


#### 确定是否业务逻辑处理较慢

1. 日志中返回较慢的请求参数。单独拿出来多次执行，返回情况正常。 
2. 与第三方系统沟通，发现出现问题请求的执行逻辑耗时相对比较短，不存在执行比较慢的情况

### 请求时间究竟消耗在什么位置

整个请求过程中，网络和业务逻辑耗时都没有问题， 但是还是缺少一个环节，因为现在服务都是只负责处理业务逻辑，accept连接和读取HTTP请求的内容，都是有框架或者第三方服务完成. 在上面查找问题缺少的环节就是server端accept请求后，将请求的内容发给服务的时间。 这个部分时间不再网络传输和业务逻辑中提现。因为这一时刻网络已经建立成功， 还没有到业务里面，所以业务逻辑的日志没有

比如：

php中 nginx -> php-fpm 

golang 中的net/http中的处理逻辑 

#### 示例代码

我用golang 做了例子


服务端代码：
``` golang 

func test(w http.ResponseWriter, r *http.Request) {
	ts := time.Now().Unix()
	// 这里是业务中执行的耗时
	w.Write([]byte(fmt.Sprintf("%d", time.Now().Unix()-ts)))
}

func main() {
	// 限制golang 协程的处理能力
	runtime.GOMAXPROCS(1)

	http.HandleFunc("/", test)
	if err := http.ListenAndServe(":51511", nil); err != nil {
		fmt.Println("%s", err)
		return
	} else {
		return
	}

}

```

客户端：
``` golang 
func main() {

	var exitWait sync.WaitGroup
	for idx := 0; idx < 1000; idx++ {
		exitWait.Add(1)
		go func() {
			ts := time.Now().Unix()
			res, err := http.Post("http://127.0.0.1:51511", "application/json", nil)
			if err != nil {
				panic(err)
			}
			respBody, err := ioutil.ReadAll(res.Body)
			if err != nil {
				panic(err)
			}
			sub := time.Now().Unix() - ts

			fmt.Println("request time(s):", sub, "server handle time(s):", string(respBody))
			exitWait.Done()
		}()
	}

	exitWait.Wait()
}
```


通过执行发现：
服务端执行在毫秒级别，但是整个请求耗时是在10s

![内存组织图](/img/http_request_slow_20200807.png)


