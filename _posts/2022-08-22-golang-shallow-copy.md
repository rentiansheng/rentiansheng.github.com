---
layout: post
title: golang bytes.Buffer 数据浅拷贝带来的问题
category: [golang]
tags: [golang]
---
### 背景

项目中定时任务获取数据，在执行过程中未对cache 的内容做变更，但是值发生了变化。 

最后定位是浅拷贝造成，在做数据交互过程中用到bytes.Buffer 做缓存区用来，包含read,write。 为了性能考虑， 缓冲区会重复使用。
同时使用byte.Buffer 和 decode，主要浅拷贝的问题. eg: 自定义json,mysql的Marshal等。 


常见的常见如下：

```go
package main

import (
	"bytes"
	"encoding/json"
	"errors"
	"fmt"
	"reflect"
	"unsafe"
)

 

type buf *bytes.Buffer

type tBytes struct {
	Name    string     `json:"name"`
	// 浅拷贝
	Config  bytesType  `json:"config"`
    // 深拷贝
	Config1 bytesType1 `json:"config1"`
}

type bytesType []byte

func (b *bytesType) UnmarshalJSON(data []byte) error {
	if b == nil {
		return errors.New("json.RawMessage: UnmarshalJSON on nil pointer")
	}
	*b = (bytesType)(data)
 	return nil
}

type bytesType1 []byte

func (b *bytesType1) UnmarshalJSON(data []byte) error {
	if b == nil {
		return errors.New("json.RawMessage: UnmarshalJSON on nil pointer")
	}
	*b = append((*b)[0:0], data...)
	return nil
}

func main() {
	jsonStr := `{"name":"marshal byte test", "config":{"aa":1}, "config1":{"aa":1}}`
	buf := bytes.NewBuffer([]byte(jsonStr))
	content := tBytes{}
	if err := json.Unmarshal(buf.Bytes(), &content); err != nil {
		fmt.Println("Unmarshal", err.Error())
		return
	}
	// 输出结果： before reset buffer marshal byte test {"aa":1} {"aa":1}
	fmt.Println("before reset buffer", content.Name, string(content.Config), string(content.Config1))
	buf.Reset()
	jsonStrNew := `{"name":"marshal byte test", "config":{"aa":3}, "config":{"aa":3}}`
	buf.Write([]byte(jsonStrNew))
	// content 没有做任何修改，但是值已经变化未最新buffer的内容
    // 输出结果：after reset buffer marshal byte test {"aa":3} {"aa":1}
	fmt.Println("after reset buffer", content.Name, string(content.Config), string(content.Config1))

	byteInfo := []byte(`12121`)[0:]
	byteInfoStr := string(byteInfo)
	sliceHeader := (*reflect.SliceHeader)(unsafe.Pointer(&byteInfo))
	stringHeader := (*reflect.StringHeader)(unsafe.Pointer(&byteInfoStr))
    // byteInfo 与byteInfoStr 的data 字段地址不同，数据没有关联
    // 824634990083 824634990088
	fmt.Println(sliceHeader.Data, stringHeader.Data)
	byteInfo[0] = '9'
    // 对byteInfo 修改，无法影响到强制转换后byteInfoStr
    // 92121 12121
	fmt.Println(string(byteInfo), byteInfoStr)

	// 注意slice 与 array 的差异, array 对象不受影响， 将array转换到reflect.StringHeader 也深拷贝（是编译器实现？） 
	zeroCopyByteInfo := []byte(`12121`)[0:]
	zeroCopySliceHeader := (*reflect.SliceHeader)(unsafe.Pointer(&zeroCopyByteInfo))
	zeroCopyStringHeader := reflect.StringHeader{
		Data: zeroCopySliceHeader.Data,
		Len:  zeroCopySliceHeader.Len,
	}
	zeroCopyByteInfo[0] = '8'
	zeroCopyConv := *(*string)(unsafe.Pointer(&zeroCopyStringHeader))
	// 82121
	fmt.Println(zeroCopyConv)
}
 
```