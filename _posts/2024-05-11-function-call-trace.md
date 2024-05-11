---
layout: post
title: 通过LSP分析代码函数调用链路
category: ["code anaylsis"]
tags: ["ast","code status anaylsis","函数调用链路"]
---


# 背景
在衡量测试质量时候，需要考虑核心链路是否被覆盖。避免核心链路出现问题。
需要通过入API 路由找到所有被调用的含漱液。 

目前函数调用链数实现方案：

1. 代码静态分析
2. 日志分析

两种方法各有缺点，链路的准确性及完整性难以保证。
同时采用AST 分析源码得到代码抽象语法树，
通过对代码文件的static anaylsis找到函数定义及函数调用位置。 
直接基于代码static code anaylsis分析缺少上下问信息。对于动态调用无法处理，
动态调用需要通过当前package 到import 与 interface 约束签名才能找到对应的关系。否则调用链路噪点太多。

# 新思路

经过对比发现IDE(goland,vscode)中跳转是一样的。
通过查找资料发现使用的gopls工具，一个叫LSP 协议。
通过几个命令串联即可以完成任务。 

LSP 协议地址。 https://microsoft.github.io/language-server-protocol/


gopls 提供基于socket 的通信。


## LSP 用到协议

1. 用workspace_symbol找到入口函数
2. 使用semtok 获取文件语义
3. 根据semtok 找到函数调用的位置
4. 调用implementation 找具体的实现
5. 跳转2. 直到被调用的文件，不是项目内代码


### workspace_symbol:



用于在工作区中搜索符号（symbol）。符号可以是函数、变量、类等代码实体的名称。

```
gopls workspace_symbol 'UserServiceImpl.QueryUserByUserEmail'

/Users/reage/goplsDev/src/test/service/user.go:72:29-49 UserServiceImpl.QueryUserByUserEmail Method
```



### symbols:

显示所选文件的符号
```
gopls symbols  service/user.go

UserService Interface 21:6-21:17
CreateUser Method 23:2-23:12
CreateUserPretty Method 24:2-24:18
DeleteUser Method 27:2-27:12
ModifyUser Method 25:2-25:12
ModifyUserPretty Method 26:2-26:18
QueryUser Method 22:2-22:11
QueryUserByUserEmail Method 28:2-28:22
UserServiceImpl Struct 31:6-31:21
departmentRpo Field 33:2-33:15
repo Field 32:2-32:6
NewUserServiceImpl Function 36:6-36:24
(*UserServiceImpl).QueryUser Method 40:29-40:38
```

### implementation:

显示所选标识符的实现， 跳转到实现
```
gopls implementation  test/role.go:10:11

/Users/reage/goplsDev/src/test/service/role.go:14:6-21
```

### definition:

显示所选标识符的声明， 转到定义

```
gopls definition test/service/role.go:25:35
/Users/reage/goplsDev/src/test/define/role.go:7:6-14: defined here as type RoleInfo struct {
    Id       uint64 `json:"id"`
    RoleName string `json:"role_name"`
    RoleDesc string `json:"role_desc"`
    Status   uint8  `json:"status"`
}


```



### semtok:

显示指定文件的语义标记， 文件中不同位置的语义信息，用来做高亮显示

```
gopls semtok  test/service/role.go

/*⇒7,keyword,[]*/package /*⇒7,namespace,[]*/service

/*⇒6,keyword,[]*/import (
"context"/*⇐7,namespace,[]*/

	"github.com/rentiansheng/test/define"/*⇐6,namespace,[]*/
	"github.com/rentiansheng/test/repository"/*⇐10,namespace,[]*/
)
```






### highlight:

显示所选标识符的突出显示, 找到被引用的位置

```
gopls highlight  test/service/role.go:28:9
 /Users/reage/goplsDev/src/test/service/role.go:28:2-10
 /Users/reage/goplsDev/src/test/service/role.go:29:26-34
```
