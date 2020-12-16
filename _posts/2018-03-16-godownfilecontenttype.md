---
layout: post
title: golang 下载文件格式错误
category: [golang]
tags: [golang, 文件下载]
---


### 场景

最近在使用golang做一个cmdb，系统中有大量的数据需要导出下载。
在功能开发完成后， 系统在开发和测试环境运行正常。但是部署到环境后，
在下载数据的时候，文件下载下来会变成zip或者没有后缀等两种情况。

### 问题分析

文件下载格式不正确，首先，想到的是http response的header中的
content-type格式不对， 使用curl -v 请求后发现返回数据如下

{% highlight curl %}
> POST /hosts/export HTTP/1.1
> Accept: */*
> Cache-Control: no-cache
> Content-Length: 246
> content-type: multipart/form-data; 
>
< HTTP/1.1 100 Continue
< HTTP/1.1 200 OK
< Date: Wed, 14 Mar 2018 09:49:55 GMT
< Content-Type: application/zip
< Content-Length: 7795

{% endhighlight %}

通过curl 调用确认了是content-type的问题。
看了golang关于http package的介绍没有设置mime.type的地方。
因为go语言使用的是系统的mime，不像nginx，apache有自己的mime配置文件。 
所以使用golang 的http package功能在不同系统上由于系统的mime.types
差异影响系统功能。 系统的mime.types在/etc/mime.types大家可以自由查看


### 解决方法

1. 更新系统的/etc/mime.types

2. 通过代码解决
 
   熟悉HTTP协议的同学应该知道，HTTP response header中的content-type
   是可以指定下载文件的格式。
   虽然相信golang作为一门优秀的语言一定会按照HTTP协议实现，
   但是还是翻了下源码，
   http.ServeFile -> serveFile-> serveContent
   
   {% highlight code %}

   func serveContent(w ResponseWriter, r *Request, name string, modtime time.Time, sizeFunc func() (int64, error), content io.ReadSeeker) {
    ...... 
    ctype := w.Header().Get("Content-Type")
   	if !haveType {
   		ctype = mime.TypeByExtension(filepath.Ext(name))
   		if ctype == "" {
   			// read a chunk to decide between utf-8 text and binary
   			var buf [sniffLen]byte
   			n, _ := io.ReadFull(content, buf[:])
   			ctype = DetectContentType(buf[:n])
   			_, err := content.Seek(0, io.SeekStart) // rewind to output whole file
   			if err != nil {
   				Error(w, "seeker can't seek", StatusInternalServerError)
   				return
   			}
   		}
   		w.Header().Set("Content-Type", ctype)
   	} else if len(ctypes) > 0 {
   		ctype = ctypes[0]
   	}
   
     ....
   } 

   {% endhighlight  %}
   
   
### HTTP头部的设置规则
 
 
 
 
 
Content-Type: application/vnd.ms-excel   #excel 2003-2007的格式

Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet  #excel 2007以后的格式
 
 
 
