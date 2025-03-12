---
layout: post
title:  go  插桩技术
category: ["go"]
tags: [  ""]
---

## 1.插桩技术
一种在程序执行过程中插入额外代码的技术，通常用于性能分析、日志记录、调试和安全监控等场景。它可以在程序的不同阶段（编译期、链接期、运行时）进行插入，具体方式包括手动插桩、编译器插桩、动态插桩等。

### 1.1  插桩技术
#### 1.1.1 静态插桩
在编译期或链接期，直接修改源代码或二进制文件，在关键函数或代码路径插入额外的分析代码。例如：

1. 编译期插桩：通过修改源代码或借助编译器提供的功能（如 GCC 插桩 -finstrument-functions）在函数入口和出口插入分析代码。 https://github.com/qiniu/goc https://github.com/alibaba/opentelemetry-go-auto-instrumentation
2. 链接期插桩：使用链接器（如 LLVM Pass、Go 的 -cover 选项）修改目标文件或可执行文件，插入额外代码。  https://github.com/golang/go/blob/master/src/cmd/cover/cover.go
#### 1.1.2. 动态插桩
在程序运行时，利用动态代码修改技术，在目标程序中插入代码，常见方法包括：

1. 使用调试 API（如 ptrace，GDB 插桩）
2. 使用 JIT 代码重写（如 DynInst，Golang runtime 插桩） https://medium.com/kokster/writing-a-jit-compiler-in-golang-964b61295f
3. 二进制修改（如 frida、Pin、eBPF 在内核态/用户态插桩）  github.com/cilium/ebpf


### 1.2 golang 插桩技术应用及选择
主要有浸入式和非侵入。 
浸入式， 比如：metrics, tracing, logging
非侵入式，  比如： go test 代码覆盖率

注意， 现在go 所谓非侵入式的插桩， 都是在编译期间插入的， 也就是静态插桩。

## 2. golang 插桩技术应用

### 2.1 浸入式插桩

- metrics
```go
	defer func() {
		metrics.USS().Service().Download(ctx, startT, "service")
	}()
```
- tracing
```go
    span, ctx := tracing.StartSpanFromContext(ctx, "service")
    defer span.Finish()
```
- logging
```go
    log.Info("service", "download", "success")
```

### 2.1 golang 非侵入式

golang 非侵入式插桩， 就是在编译期间插入的， 也就是静态插桩。
```go
 go test  cover --func=coverage.out 
```

### 2.2 golang 市场常见的插桩工具

golang 场景的非侵入式插桩工具，是在go build 前， 通过对源代码预处理（Preprocess）和代码注入（Instrument）完成。  
-  [goc](https://github.com/qiniu/goc)
- [opentelemetry](https://github.com/alibaba/opentelemetry-go-auto-instrumentation)

因为目前在go 文件编译二进制的过程中，未暴露出来任何hook功能， 
golang 具体编译如下: 
1. 源码解析：Golang编译器会先解析源代码文件，将其转化为抽象语法树（AST）。
2. 类型检查：解析后会进行类型检查，确保代码符合Golang的类型系统。
3. 语义分析：对程序的语义进行分析，包括变量的定义和使用、包导入等。(SSA) 
4. 编译优化：将语法树转化为中间表示， 进行各种优化，提高代码执行效率。
5. 代码生成：生成目标平台的机器代码。
6. 链接：将不同包和库链接成一个单一的可执行文件。


### 2.3 如何实现编译前插桩

通过 AST 解析代码结构，将插桩代码注入到源代码中，然后再进行编译。 

```go

var _importPath = "github.com/rentiansheng/instrumentation-func/demo_log"
//  gen import  
func ImportFunctrace(f *ast.File) {
    importFlag := false
    for _, item := range f.Imports {
        if item.Path.Value == _importPath {
            importFlag = true
            break
        }
    }
    // 如果没有导入包,  导入包
    if !importFlag {
        astutil.AddImport(r.fset, f, _importPath)
    }
}

// 生成defer 代码
func genDefer(f *ast.File, packageName, fn string ) *ast.DeferStmt {
    sliceParams := &ast.CompositeLit{
        Type: &ast.ArrayType{
            Elt: &ast.StringType{ // 空接口
                Methods: &ast.FieldList{
                    List: []*ast.Field{},
                },
            },
        },
        Elts: elts,
    }
	callExpr := &ast.CallExpr{
        Fun: &ast.CallExpr{
            Fun: &ast.SelectorExpr{
                X:   ast.NewIdent("demo_log"),
                Sel: ast.NewIdent("Duration"),
            },
            Args: []ast.Expr{
                ast.Expr(&ast.BasicLit{Kind: token.STRING, Value: packageName}),			    
                ast.Expr(&ast.BasicLit{Kind: token.STRING, Value: fn}),
            },
		},
    }
    return &ast.DeferStmt{
         Call: callExpr,
    }
}

func genFile(f *ast.File) {
	
    // 插入defer函数
    for _, item := range f.Decls {
        funcDel, ok := item.(*ast.FuncDecl)
        if !ok {
            continue
        }

        deferStmt := genDefer(f, f.Name.Name, funcDel.Name.Name)
        funcDel.Body.List = append([]ast.Stmt{deferStmt}, funcDel.Body.List...)
    }
	
}


``` 





