---
layout: post
title:  区分测试工具
category: ["go"]
tags: [""]
---

### 背景

在测试管理平台中，不同测试工具发起的 HTTP 请求需要通过 HTTP Header 携带特定标签，以便平台识别和分类处理。
如何在不修改测试代码的前提下，自动为所有 HTTP 请求注入自定义 Header 成为一个关键需求。

#### 关键技术结介绍

http proxy 工作原理：

正向 HTTP 代理（Forward Proxy）是一种位于客户端和目标服务器之间的代理服务器。
客户端（如浏览器、App、测试程序）先把请求发送给代理服务器，再由代理服务器转发到目标网站，然后把响应返回给客户端。

过程：

- 所有请求先发到代理服务器。
- 代理服务器替你访问目标网站。
- 目标网站的响应返回到代理服务器，再由代理服务器转发给你。

通俗讲类似货物发送， 商家-》买家， 比较近的时候，直接商家-》买家，太远了就给快递，负责中转。

### 1. 方案原理：
   通过设置本地 HTTP 代理服务器（如 127.0.0.1:8888）；
   所有流量（HTTP/HTTPS）都通过该代理转发；
   代理服务器拦截所有请求，在请求头部自动注入自定义 Header；
   对 HTTPS 需启用中间人攻击（MITM）模式，便于读写加密流量。
### 2. 关键步骤：
   启动本地代理服务；
   设置环境变量HTTP_PROXY和HTTPS_PROXY指向代理服务地址；
   在代理服务内，对所有出站请求自动注入如X-Custom-Header: testops-auto；
   支持所有语言的测试代码透明接入，无需修改测试代码本身。

### 3. 优缺点：
   优点：
   透明注入 Header，用户无感知；
   支持任何语言（只要支持环境变量代理）。
   缺点：
   HTTPS 需开启 MITM，可能引发证书信任问题；
   适合于测试环境，需注意环境隔离和安全。
   用户自定义http client 头中的 net.Transport, 未设置代理相关配置，会失效




### 4. 关键代码：

#### 1.go HTTP 代理服务

```go
package main

import (
   "log"
   "github.com/elazarl/goproxy"
   "net/http"
)

func main() {
   proxy := goproxy.NewProxyHttpServer()
   proxy.OnRequest().HandleConnect(goproxy.AlwaysMitm)
   proxy.OnRequest().DoFunc(
   func(r *http.Request, ctx *goproxy.ProxyCtx) (*http.Request, *http.Response) {
   r.Header.Set("X-Custom-Header", "test-platform")
   return r, nil
   })
   proxy.Verbose = true
   log.Fatal(http.ListenAndServe(":8888", proxy))
}
```





#### 2.配置环境变量：

```bash

export HTTP_PROXY=http://127.0.0.1:8888
export HTTPS_PROXY=http://127.0.0.1:8888

```



### 5. 自定义net.Transport 处理异常场景

没有设置Proxy或者 Proxy = nil 字段，则不会走代理。

```go 
tr := &http.Transport{
   Proxy: http.ProxyFromEnvironment, // 这行必须有
   // 其他配置 ...
}

```

