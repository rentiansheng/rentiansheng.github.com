---
layout: post
title:  golang 动态结构体
category: [golang 动态结构体]
tags: [go]
---

###  背景

语言： golang

目前项目允许业务自定自己业务下私有字段。 造成统一类型实例数据，在不同的业务下字段是不一样。 在实际处理中无法使用struct来定义。
当实例数据在不同服务之间数据传递， 只能用key-val结构(map[string]interface{})来描述，在实际处理中非常不变，
需要大量判断数据类型和类型转换代码，整体使用起来比较麻烦。

eg: 
```go

    inst := make(map[string]interface{},0)
    err := json.Unmarshal([]byte(....), &inst)
    if err != nil {
        return 
    }
    var name string 
    if iName := inst["name"]; iName != nil {
        id, ok := inst["id"].(string)
        if !ok {
           return errors.New("invalid")
        }
    }
   


```


### 如何解决大量类型转换和判断数据类型这个问题？

struct 在我们使用的时候之所以方便，简洁就是在做反序列的时候根据结构的定义字段和类型。 所以在使用的可以不用在做类型判断和数据校验
如果可以做一个dynamic struct，这个struct字段是根据执行逻辑动态调整字段和字段类型即可。

### 最终实现

最终的实现定一个DStruct 结构， 给DStruct结构实现基于byte stream json 反序列， 利用golang可以自定义struct UnmarshalJSON的特性，
当做json Unmarshal调用DStruct 根据用户定义的配置项目反序列化。 [项目地址](https://github.com/rentiansheng/dstruct) 

eg: 

```golang

 
	testStrJSONBytes := `{"int":1,"str":"str","bl":true, "arr_str":["1","2"],"arr_int":[1,2], "map":{"a":1,"b":64}, ` +
		`"test_struct":{"str":"string", "int":1},"interface":{"str":"string", "bl":true}}`
		// define a dynamic structure
	ds := &DStruct{}
	var i interface{}
	// define the fields of the dynamic structure
	ds.SetFields(map[string]reflect.Type{
		"int":         TypeInt,
		"str":         TypeString,
		"bl":          TypeBool,
		"arr_str":     TypeArrayStr,
		"arr_int":     TypeArrayInt,
		"map":         reflect.TypeOf(map[string]int64{}),
		"interface":   reflect.TypeOf(i),
	})
	// Unmarshal dynamic structure, you can directly use golang official
	err := json.Unmarshal([]byte(testStrJSONBytes), ds)
	if err != nil {
		t.Error(err.Error())
		return
	}
    intVal, err := ds.Int64("int")
    if err != nil {
		fmt.Error(err.Error())
		return
	}
		fmt.Println("int test, err: ", err, "val: ", intVal)
	str, err := ds.Str("str")
	if err != nil {
		fmt.Error(err.Error())
		return
	}
	fmt.Println("str test, err: ", err, "val: ", str)
	exists, err := ds.Value("interface", &i)
	if err != nil {
		fmt.Error(err.Error())
		return
	}
	fmt.Println("exist field: ", exists, "val: ", i)
```



