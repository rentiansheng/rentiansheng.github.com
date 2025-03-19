---
layout: post
title:  Golang  精准测试-找到代码修改影响范围1
category: ["go"]
tags: [""]
---

### 背景

 我们会发现业务修改的代码分散多个文件，甚至多个服务中，在测试过程中，我们很难确定影响到的上游或者那些case 可以用，
 这时候我们需要找到修改代码的影响范围和上游调用函数，以便我们可以更好的进行测试。


### 主要思路

我们讲仓库的代码符号化，提取出来结构，interface，函数，变量等信息，然后通过代码对比，找到修改的代码的影响范围。

主要是： 
1. 环境准备：执行依赖，比如：go mod tidy, wire gen
2. 代码解析： SSA及AST
3. 代码符号化： 描述结构体，函数，静态调用等信息

### 代码实现

####  load 仓库代码

使用 package.Load 加载代码， 用获取AST及SSA 信息。

[入口代码](https://github.com/rentiansheng/fc/blob/master/code_anaylsis/scan.go#L108)
```go
initial, err := packages.Load(s.cfg, s.Dir+"/...")
```

#### 分析函数body 中调用新

主要对ssa.Function 中的block 进行处理， 有以下需要注意的:
1. 这里需要注意匿名函数和在参数中的匿名函数
2. 主要不同的调用方式， 比如通过通过interface 和结构体。 目前通过interface 分析还不够完善，需要进一步完善。（后续会支持根据依赖注入）


[代码](https://github.com/rentiansheng/fc/blob/master/code_anaylsis/function.go#L16】

```go  
	c1, c2 := s.parserFuncBodyBlocks(ctx, fn.Blocks)
	// 如果有匿名函数处理
	for _, closuresFn := range fn.AnonFuncs {
		itemC1, itemC2 := s.parserFuncBodyCode(ctx, closuresFn)
		c1 = append(c1, itemC1...)
		c2 = append(c2, itemC2...)
	}
	return c1, c2
```
 
