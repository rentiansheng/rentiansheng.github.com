---
layout: post
title:   go exec.Command使用pipe做数据交换遇到的问题
category: [go]
tags: [go]
---


### 背景

使用exec.Command 实现syscall.ForkExec功能，为了让master与child数据交互的能力，
最终exec.Command.ExtraFiles 来传递pipe 的fd 来实现，
但是实际使用遇到几个奇怪的问题，感觉和系统实现的有些差别。
使用不好会卡到数据交互。比如卡到数据的读，读到数据为空

注意： 
- ioutil.ReadAll 需要读到EOF 才会结束， 如果使用pipe，就需要关闭 write 通道

### 例子
也不知道怎么描述， 直接上代码

```go
package main
import (
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
)
func execChildFunc() bool{
	aa := os.Getenv("aa")
	if aa != "" {
		file := os.NewFile(3, "")
		readFile := os.NewFile(4, "")
		file.Write([]byte("test success"))
		paramBytes, _ := ioutil.ReadAll(readFile)
		file.Write(paramBytes)
		return true
	}
	return false
}
func pipe() (*os.File,*os.File,*os.File,*os.File, error ) {
	masterRead, childWrite, err := os.Pipe()
	if err != nil {
		return nil, nil, nil, nil, err
	}
	childRead, masterWrite, err := os.Pipe()
	if err != nil {
		return nil, nil, nil, nil, err
	}
	return  masterRead, childWrite,childRead, masterWrite, nil
}
func main() {
	// fork
	if execChildFunc() {
		return
	}
	self, err := os.Executable()
	if err != nil {
		fmt.Println("get self path error.",err)
		return
	}
	masterRead, childWrite,childRead, masterWrite, err := pipe()
	if err != nil {
		fmt.Println("pipe init error.",err)
		return
	}
	masterWrite.Write([]byte(" test master params"))
	execCmd := exec.Command(self, os.Args...)
	execCmd.Env = append(os.Environ(), "aa=2222222")
	execCmd.Stderr = os.Stderr
	execCmd.Stdout = os.Stdout
	execCmd.ExtraFiles = []*os.File{childWrite,childRead}
	err = execCmd.Start()
	if err != nil {
		fmt.Println(err)
		return
	}
	// 必须关闭，会卡到masterRead read 位置
	childWrite.Close()
	// 关不关闭对流程没有影响
	childRead.Close()
	// 不关闭  会卡子进程read 数据， 子进程无法退出
	masterWrite.Close()
	if err :=execCmd.Wait(); err != nil {
		fmt.Println(err )
	}
	rsp, err := ioutil.ReadAll(masterRead)
	if err != nil {
		fmt.Println("read child result error.",err)
		return
	}
	fmt.Println("child result : ",string(rsp))
}
```