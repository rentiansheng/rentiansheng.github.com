---
layout: post
title:  golang 泛型
category: [golang泛型]
tags: [go]
---



###  背景

目前所在项目是关于资产管理，经常会有更重数据做各种操作，比如，各类数值型Join, Sort, Unique等很多操作。  

由于在目前的golang 版本不支持多态， 只能人工的方式来实现。效果如下：
![人工泛型](https://lh3.googleusercontent.com/-5Vz_puoHe7o/WMJukhX878I/AAAAAAAADRI/9nn3cse3gkYrZ--aVfWaofUFGaRb49MtACLcB/s1600/Animation.gif)
（图片来源）https://groups.google.com/g/golang-nuts/c/j_3n5wAZaXw/m/YkOdbCppAQAJ


### 泛型实现

- 多态
- 类型推导
- 操作符重载

### golang 实现

golang 目前完成的主要是类型推导


### 如何使用golang

##### 编译支持golang的版本

```angular2html
git clone https://github.com/golang/go
git checkout dev.go2go
cd src
./all.bash 

go tool go2go run xx.go2
```


##### 例子

(https://go2goplay.golang.org/)[官方实例]


数据类型的Join. file name: intJoin.go2

```go
package main
import (
	"fmt"
	"strings"
)

func main() {

	str := Join([]int64{1,2,3,4});
	fmt.Println("int64", str)
	str = Join([]int32{1,2,3,4});
	fmt.Println("int32", str)

	str = JoinNumber([]int64{1,2,3,4});
	fmt.Println("int64", str)
	str = JoinNumber([]int32{1,2,3,4});
	fmt.Println("int32", str)

	// NOTICE: ERROR. 类型不匹配时候的问题
    /* 
	str = JoinNumber([]string{"1","2","3","4"});
	fmt.Println("int32", str) 
	*/
}

func Join[T any](arr []T) string {
	str := ""
	for _, v := range arr {
		str = fmt.Sprintf("%s,%v", str,v)
	}

	return strings.Trim(str, ",")
}

type numeric interface {
	type int, int8, int16, int32, int64, uint, uint8, uint16, uint32, uint64, float32, float64
}

func JoinNumber[T numeric](arr []T) string {
	str := ""
	for _, v := range arr {
		str = fmt.Sprintf("%s,%v", str,v)
	}

	return strings.Trim(str, ",")
}
```



数组结构体多态实例， file: struct.go2

```go
package main

import ("fmt")

type Pair[T1 , T2 any] struct {
    ID T1
    Val T2
}

type MapArr[T1, T2 any] struct {
    s []Pair[T1, T2]
}

func (m * MapArr[T1,T2]) Set(k T1, v T2) {
    m.s = append(m.s, Pair[T1,T2]{k,v})
}

func (m * MapArr[T1,T2]) Print()  {
    for _, p := range m.s {
        fmt.Printf(" info:(id: %s, val:%v)", p.ID, p.Val)
    }

    fmt.Printf("\n")
}


func main() {
  kvArr := &MapArr[string, int]{}
  kvArr.s  = []Pair[string, int]{}
  kvArr.Set("k1", int(1))
  kvArr.Set("k2", int(2))
  fmt.Printf("str,int")
  kvArr.Print()

  kvArr1 := &MapArr[string, string]{}
  kvArr1.s  = []Pair[string, string]{}
  kvArr1.Set("k1", "v1")
  kvArr1.Set("k2", "v2")
  fmt.Printf("str,str")
  kvArr1.Print()

}

```