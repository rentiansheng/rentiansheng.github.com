---
layout: post
title:  Golang  精准测试-找到两次部署差异协助测试
category: ["go"]
tags: [""]
---

### 背景

在测试过程中，同一需求可能会经历多次部署，但部分部署的代码改动较小。如果每次都进行全量测试，成本较高。因此，我们希望通过代码分析来判断之前的测试或审查是否仍然有效。

在代码对比过程中，需要忽略一些不影响逻辑的改动，例如：

- 代码中的注释
- 代码中的空格和换行符

目标是实现精准测试，重点关注本次修改所影响的功能。对于已测试过的部分，可以降低测试关注度。例如：

- 识别相关的测试用例（自动化、手动测试、流量回放）并重新执行或生成新的测试用例
- 合并覆盖率数据，确保改动代码已被测试
提供未改动代码的函数测试功能合并，指导测试过程，达到"改过即测过"的目标。

*注意：* 修改一行代码，不是仅需要测试一行代码，他有一个影响范围，每一个函数是一个独立的功能，所以我们需要测试的是一个函数。 这里可以和增量代码结合

### 主要对比思路

- AST 解析：通过 go/parser 解析 Go 代码，生成 AST（抽象语法树）。
- 获取函数信息：提取所有函数的签名及其 AST 结构。
- 对比函数：
    1. 通过函数签名匹配相同名称的函数 
    2. 忽略无关信息（如注释、空格、换行、位置信息, 调用其他函数等）
    3. 对比 AST 结构，判断函数逻辑是否一致
- 识别影响范围：
    1. 找出修改过的函数，并确定其影响的测试范围
    2. 对未修改的函数复用测试结果，减少回归测试开销

*注意：*  这里需要忽律一些对比字段值，比如： 位置信息，第三方调用的函数(AST 将所有信息关联起来，如果存在调用，本地AST 其他对象，这里也会受影响的)

###  效果

```go

// file1.go 内容
package code
type  methodFunc struct {}
func (m *methodFunc) same1()   { /*xxxx*/}
func (m methodFunc) same2()  {var a int; _ = a}
func (m methodFunc) diff1()  {}
func (m methodFunc) diff2()  {}
func  same1()  {}
func  same2()  {}
func diff1()  {}
func diff2()  {}

// file2.go 内容
package code
type  methodFunc struct {}
func (m methodFunc) same2()  {var a int; _ = a}
func (m *methodFunc) same1()  {}
func (m methodFunc) diff2()  { {var a int; _ = a}}
func (m methodFunc) diff1()  { {var a int; _ = a}}
func  same1()  {}
func  same2()  {}
func diff2()  { {var a int; _ = a}}
func diff1()  { {var a int; _ = a}}
```

对比结果如下：
- methodFunc.same1,methodFunc.same2,same1,same2 等四个函数在file1.go 和file2.go 内容是一样的
- methodFunc.diff1,methodFunc.diff2,diff1,diff2 等四个函数在file1.go 和file2.go 内容是不一样的

根据结果我们发现，注释和位置信息不影响结果判断。 


### 具体是实现 

具体代码在: [code compare](https://github.com/rentiansheng/fc/tree/master/code)





