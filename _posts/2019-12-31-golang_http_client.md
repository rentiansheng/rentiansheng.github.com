---
layout: post
title:  golang  http client 遇到的问题
category: [golang, http/client]
tags: [golang]
---

记录最近在使用golang http client 遇到的问题


### response body 乱码

服务线上没有问题， 但是在开发阶段调用第三服务出现返回无法解析的情况。 

最后发现是数据被压缩造成。

golang HTTP client 在收到数据的时候，在http response 在Context-Encoding=[giz\|deflate\|br]的时候也不去解压response body的内容，需要业务方自己去解压。 注意在golang自带有关压缩的实现，没有看到与br压缩的相关类库。




### http client 发送请求连接不上服务

在terminal,浏览器发送请求都可以正常访问，但是在golang服务中的HTTP 请求返回连接不上服务(i/o timeout)。这种情况查看下使用的HTTP 的client中的transport的Proxy 属性是不是给设置为nil，golang的http package中有一个叫ProxyFromEnvironment方法。改方法是从环境变量中获取配置的代理信息的。如果你需要使用系统配置的Proxy可以使用http.ProxyFromEnvironment方法,否则可以实现自己的proxy方法。

