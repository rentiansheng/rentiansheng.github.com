---
layout: post
title:  Golang  取地址对编译速度的影响
category: ["go"]
tags: [""]
---

### 背景

  最近在在做关于流量维度的执行流水了， 需要对代码做插桩， 在业务的代码每一个执行分支（block）的起始位置做插桩。用来记录执行的时候，不同block 前后顺序。 
  我们在对项目代码做了插桩之后，go build 编译代码的时候非常慢。 从原来的分钟级别变为十分钟。


### 问题定位
通过go build -work -x 查看编译过程。  go build 输出是没有时间输出。 我这里时间是公司Devops 平台的。  
1.先定位到编译变慢行为, 我这里找到的是在compile  阶段. 
```bash
/usr/local/go/pkg/tool/linux_amd64/compile -o $WORK/b965/_pkg_.a -trimpath "$WORK/b919=>" -p package -lang=go1.15 -complete -buildid xxx -goversion go1.15.15 -D "" -importcfg $WORK/b919/importcfg -pack -c=4 ./cover.go
````
2. 通过 compile  命令中的-p 参数， 可以看到是编译的package
3. 分析这个package 下的代码
发现是插桩后生成代码， 这里主要记录文件与block的关系。 由于业务代码比较大。这个文件在6m左右；
```azure
var  allFiles = []string{.......}
var blocks = []struct{
    Loc ....
    File *string
} {
    {loc1,&allFiles[n]},{loc2,&allFiles[n]},{loc3,&allFiles[n]}
}

```
4.通过分析代码，这里为了节省内存，对所有的文件名都是通过地址引用过去的
