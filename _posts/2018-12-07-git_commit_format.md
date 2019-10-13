---
layout: post
title:  git comMit messge 消息格式规范
category: [git]
tags: [git]
---
根据外部文档总结出来git commit 提交的格式规范

# git commit messge 消息格式
```
  type:messsge issue 
```
样例

```
  # 新加一个文档 
  git commit -m "docs: add readme document issue #1 #2"  readme.md
```

## type 取值范围

|标记|含义|加入版本|
| ------------ | ------------ |---------|
|feature |新功能|v1|
|fix|错误修复|v1|
|docs| 文档更改|v1|
|style|（格式化，缺少半冒号等;没有代码更改）|v1|
|refactor| 代码重构重构|v1|
|test| 添加缺失的测试，重构测试;没有生产代码更改|v1|
|chore| 构建脚本，任务等相关代码|v1|
|depend |依赖的第三方代码|v2|
|lib|     公共类库代码|v2|
|define|  公共变量定义|v2|    
|merge|不同分支之间的代码合并, issue 内容可以忽略|v2|
|info|注释等描述类型内容|v3| 

## message
本次提交的描述

## issue
本次提交相关的issue ,可以有多个
