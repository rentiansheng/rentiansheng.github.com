---
layout: post
title: reflect 使用总结
category: [golang]
tags: [reflect]
---

## 1 如何确定数据类型

使用 reflect.Kind 或者reflect.Type


### 1.1 reflect.Kind

go的基础类型, 类型是有限，可以被穷举。

```
type My string  kind=string
type kv map[string]interface{}  kind=map
```

### 1.2  reflect.Type

用户定义的类型, 内置类型及用户type 定义类型及闭包类型，

```
type myStr string   type= main.myStr
```

## 2 如何操作数据

注意： reflect 操作需要开发去考虑边界情况，出现数据类型不匹配，越界，未初始化，非类型操作函数等情况会直接panic。

reflect 提供一系列操作基础数据的操作 比如： GetXXX,SetXXX.

reflect 对一些操作做聚合，比如：Elem(),Key(), Len(),数字操作int,int8,int16,int32,int64 使用Int(), SetInt(x)

reflect 对数据操作是一个超集合，需要根据类型使用不同函数


### 2.1 访问数据需要注意

Elem() ： 用来获取复杂数据类项目的值， 比如： 指针，slice，array, map 的Value,
Map 操作： 方法 map 的key 可以用reflect.ValueOf(kv{}).Type().Key() 来确定类型， MapKeys(), MapRange(),MapIndex() 来访问
Struct 操作： 需要注意inline 结构体及私有对象操作, 需要递归处理，无法用Field获取到对应Field


### 2.2 修改数据需要注意
CanXXX 方法提供数据类型是否配置校验
操作的对象必须是指针对象， 可以用 CanSet() 方法判读
reflect.New 出来的是指针类型，赋值时候需要注意， 
Array 对象操作需要注意，不能append。需要用index 修改
Struct 操作的时候， FIeld,FieldByIndex无法处理inline struct 字段， 私有字段不允许修改，如果需要修改可以与unsafe.Pointer
IsZero() 与IsNil() 区别。 IsZero()对所有数据类型有效，默认值及nil 为true， InNil()仅对 chan, func, interface, map, pointer, or slice 有效


## 3. 使用样例

golang 数据深拷贝的类库，支持数据自动映射。 map to struct, struct to map, struct to struct.
https://github.com/rentiansheng/mapper
