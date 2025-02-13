---
layout: post
title:  通过修改GO 源代码，支持获取goroutine id 和 协程调用链全局上下文
category: ["go"]
tags: ["go source add grammar"]
---


## 1.背景

在开发过程中，我们经常需要获取goroutine id 和全局上下文。但是go runtime 并没有提供这样的接口。我们可以通过修改go runtime 源码，添加这样的接口。

## 2.下载源代码

```shell
cd $GOPATH/src
git clone https://github.com/rentiansheng/go.git
cd go
git fetch && git checkout -f toy_context
cd src
## 编译命令
./make.bash
```

## 3.修改源代码

如果想要获取goroutine id 和 实现协程调用链全局上下文， 我们需要找到协程定义的结构及提供对应的访问API。
我们需要在协程创建的时候，自动继承父goroutine上下文信息, 同时支持设置上下文信息，或者重置上下文信息。 

协程定义在 $GOPATH/src/go/src/runtime/runtime2.go 文件中。我们需要未他加一个字段，用来存储上下文信息

协程管理在 $GOPATH/src/go/src/runtime/proc.go 文件中。我们需要在创建协程的时候将上下文信息传递给协程。


具体修改如下：
1. 在runtime2.go 文件中添加字段

```go
type g struct {
	...

	// global context 
	context map[string]any
}

```

2. 在proc.go 文件中添加上下文传递

```go
//go:nosplit
func GoID() uint64 {
    return getg().goid
}

//go:nosplit
func GoSpilt() {
    getg().context = map[string]any{}
}

//go:nosplit
func GoValue(key string) any {
    if getg().context == nil {
    return nil
    }
    return getg().context[key]
}

//go:nosplit
func GoSetValue(key string, v any) {
    if getg().context == nil {
        getg().context = map[string]any{}
    }
    getg().context[key] = v
    return
}


func newproc1(fn *funcval, callergp *g, callerpc uintptr, parked bool, waitreason waitReason) *g {
	...
	newg.context = callergp.context
    return newg
}
```


## 4.编译源代码

```shell
cd $GOPATH/src/go/src
./make.bash
```

## 5.测试example

测试样例在 $GOPATH/src/go/reage/main.go 文件中

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	fmt.Println("Hello, word", runtime.GoID())

	runtime.GoSpilt()
	runtime.GoSetValue("key", "test")
	go g()

	go gSplit()

	time.Sleep(1 * time.Second)
}

func g() {
	runtime.GoValue("key")
	fmt.Println("Hello, word", runtime.GoID(), runtime.GoValue("key"))
	go g1()
}

func gSplit() {
	runtime.GoSpilt()
	fmt.Println("Hello, word", runtime.GoID(), runtime.GoValue("key"))
	go g1()
}

func g1() {
	fmt.Println("Hello, word", runtime.GoID(), runtime.GoValue("key"))
}

```

go源代码编译成功后，运行测试样例， 具体命令如下：
```shell

cd $GOPATH/src/go/

GOROOT=/Users/tiansheng.ren/goDev/src/go  ./bin/go   run ./reage/main.go

```
输出结果如下(其中数字是协程id，每次执行会有差异)：
```shell
Hello, word 1
Hello, word 20 <nil>
Hello, word 21 <nil>
Hello, word 19 test
Hello, word 22 test
```



