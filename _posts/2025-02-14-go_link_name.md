---
layout: post
title:  func 绑定
category: ["go"]
tags: [  "go linkname"]
---

# 1.go linkname 介绍

//go:linkname 是 Go 语言中的一个特殊编译指令，用来在编译期间将一个标识符链接到另一个标识符，特别是用于访问未公开（未导出）的标识符。这种指令允许在 Go 代码中跨包访问原本无法直接访问的非公开成员，通常用于优化或者底层的调试操作。

```go
//go:linkname <新标识符> <包名>.<原始标识符>
```

- <新标识符>：指定你在当前包中希望使用的名称。
- <包名>.<原始标识符>：指定要链接的目标符号，通常是其他包中的私有成员（如非公开的结构体或函数）。
# 2.使用场景


- 访问私有符号：某些情况下，Go 的标准库或第三方库中的标识符是未公开的，但你希望访问它们。这时可以使用 //go:linkname 来绕过 Go 的访问控制限制。
- 跨包共享实现：有时你希望在多个包之间共享内部实现，//go:linkname 可以帮助你在不暴露 API 的情况下实现这一目标。
- 性能优化或低级调试：如果你需要访问底层的实现，或者调试一些低级别的行为，可以使用此指令来确保可以直接访问特定的符号。
# 3. 注意事项

live 环境不要使用， 仅用于做测试， 使用会被go 官方嫌弃
- 仅限编译期使用：//go:linkname 只在编译时有效，它不会在运行时对代码产生任何影响。
- 绕过访问控制：它绕过了 Go 语言的访问控制（即大写字母表示导出、以小写字母开头表示未导出）。这意味着你可以访问通常无法访问的私有符号，但这也可能导致不安全的代码实践。
- 不推荐用于生产环境：虽然 //go:linkname 是 Go 编译器支持的，但它通常不适合生产环境中的日常使用，因为它违反了 Go 语言的封装和模块化原则。


# 4.示例


参考： https://github.com/rentiansheng/go/blob/release/src/routine/routine.go
```go
type goroutineDoneFn func() <-chan struct{}

//go:linkname Id runtime.GoID
func Id() uint64

//go:linkname Split runtime.GoSplit
func Split()

//go:linkname Value runtime.GoValue
func Value(key string) any

//go:linkname Set runtime.GoSetValue
func Set(key string, v any)

//go:linkname WithDone runtime.GoWithDone
func WithDone(fn goroutineDoneFn)

//go:linkname IsDone runtime.GoIsDone
func IsDone() bool

```
