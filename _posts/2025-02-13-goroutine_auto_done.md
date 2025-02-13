---
layout: post
title:  通过修改GO 源代码，支持goroutine  自动done
category: ["go"]
tags: ["go source add grammar", "goroutine schedule auto check is done"]
---


## 1. 原理

在goroutine 执行过程中，在做协程调度的时候，我们需要判断协程是否执行完成。如果没有执行完成，我们需要继续调度。如果执行完成，抛出异常。


## 2.实现 

### 2.1 在runtime2.go 文件中添加字段 用来帮助调度协程是否执行完成

文件具体位置在 $GOPATH/src/go/src/runtime/runtime2.go 文件中
```go

type g struct {
    ...
	//  done chan
	done goroutineDoneFn
}


type goroutineDoneFn func() <-chan struct{}


```
### 2.2 在proc.go 文件中添加控制方法和自动检查逻辑

文件具体位置在 $GOPATH/src/go/src/runtime/proc.go 文件中。我们需要在创建协程的时候，继承来自父协程的done chan。

在调用的时候，我们需要判断done chan 是否关闭。如果关闭，说明协程执行完成，发出异常。否则继续调度。

```go

/*** goroutine schedule check is done ***/

func execute(gp *g, inheritTime bool) {
    ...
    if goIsDone(gp) {
        throw("execute: goroutine is done")
    }
	...
}

/***  控制方法 ***/


//go:nosplit
func GoSpilt() {
    ...
    getg().done = nil
}

//go:nosplit
func GoWithDone(fn goroutineDoneFn) {
    getg().done = fn
}

//go:nosplit
func GoIsDone() bool {
    return goIsDone(getg())
}

//go:nosplit
func goIsDone(gp *g) bool {
    if gp.done == nil {
        return false
    }
    select {
        case <-gp.done():
            return true
        default:
    }
    
    return false
}


/***  创建协程时候，继承数据 ***/

func newproc1(fn *funcval, callergp *g, callerpc uintptr, parked bool, waitreason waitReason) *g {
	...
	newg.done = callergp.done
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

执行的时候 会抛出异常，说明协程执行完成。

 ``` 
Hello, word 1
Hello, word 18 test
Hello, word 19 <nil>
Hello, word 20 <nil>
fatal error: execute: goroutine is done

 ```

