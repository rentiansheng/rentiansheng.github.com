---
layout: post
title:  net/http timeout awaiting response headers
category: [golang]
tags: [golang net/http question]
---


### 前提

在做系统压测的是发现有部分接口，返回 **net/http: timeout awaiting response headers** 错误，

### 分析

看到错误首先想到的是http请求超时， 修改http client的timeout发下没有任何效果。

但是这个错误就是http client 超时引起。是http client 在一定时间内没有返回数据，
客户端取消连接引起。 这个是由于http.Client的transport中ResponseHeaderTimeout 设置不合理引起的。


### 相关知识

net.Dialer.Timeout 限制建立TCP连接的时间
http.Transport.TLSHandshakeTimeout 限制 TLS握手的时间
http.Transport.ResponseHeaderTimeout 限制读取response 返回内容的时间
http.Transport.ExpectContinueTimeout 限制client在发送包含 Expect: 100-continue的header到收到继续发送body的response之间的时间等待。注意在1.6中设置这个值会禁用HTTP/2(DefaultTransport自1.6.2起是个特例)


### 场景重现代码


使用下面代码中的client 调用服务端即可

```golang

// server code 
func httpServer() {
	srv := &http.Server{
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
		Addr:         ":65530",
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		str := "{}"
		time.Sleep(3 * time.Second)
		w.Write([]byte(str))
	})
	go func() {
		err := srv.ListenAndServe()
		if nil != err {
			fmt.Println(err)
		}
	}()
	time.Sleep(10)

}

// client code
func httpClient() {
	for i := 0; i < 10; i++ {
		transport := &http.Transport{
			TLSHandshakeTimeout: 5 * time.Second,
			TLSClientConfig:     nil,
			Dial: (&net.Dialer{
				Timeout:   500 * time.Second,
				KeepAlive: 30 * time.Second,
			}).Dial,
			ResponseHeaderTimeout: 1 * time.Second,
		}
		c := &http.Client{
			Timeout:   10 * time.Second,
			Transport: transport,
		}
		_, err := c.Get("http://127.0.0.1:65530/aa")
		if nil != err {
			fmt.Println(err)
		}


	}

}

```

