---
layout: post
title:  web 服务error的设计
category: [web ]
tags: [design]
---


## 1. 为什么要设计系统自己的error

我们为什么要设计的系统自己的error，就为了重复造轮子？

每个语言都有自己的error系统，为什么不使用？

要回答这些问题。需要先了解error出现和使用的场景。

error 是当服务在链接第三方服务，校验数据，处理逻辑异常的时候用于提示用户，查找出错原因第一手资料。

用户的提示：简单易懂

开发人员： 需要详细的上下信息

面向第三方系统： 错误归类，判断。

所以当以错误产生的时刻需要将错误处理为面向用户，面向开发人员和第三方系统的格式。

语言本身的系统error或exception 适用于开发人员。带有很强的专业性。不适宜展示给用户或者第三方系统。 语言本的系统error或exception 也缺乏去多语言，多长场景的支持。

### 2. error 内容

error_code  与第三方系统交互实用 

error_messsage  提示用户使用

request_id  聚合日志使用

http_code  http response 状态码

### 3. error 带有的方法
 * **error  code 对应一个format 格式字符串**
 
```code

// 根据错误码和参数生成error
Errorf(code int, .... interface{}) 

// 根据错误生成error
Error(code int)

// 根据第三方系统的error code 和error message 生成error
New(code int, msg string)

// 打印日志使用
String ()  string 

// error code 
Code() int

// error message

Message() string

```
