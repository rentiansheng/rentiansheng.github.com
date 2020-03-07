---
layout: post
title:  蓝鲸配置平台-蓝鲸cmdb开发登录插件
category: [bk-cmdb]
tags: [bk-cmdb]
---


#### 前提

蓝鲸配置平台（bk-CMDB）是一个面向资产及应用的企业级配置管理平台。
是蓝鲸运维体系一部分，蓝鲸社区版本是免费，社区地址[https://bk.tencent.com/]()
并且蓝鲸很多组件已经开源，蓝鲸cmdb就是其中之一，

蓝鲸配置平台作为蓝鲸一部分登录及用户管理依赖蓝鲸登录及用户组件， 
如果药蓝鲸cmdb使用其他登录系统，就需要进行第三方开发。 



#### 准备

开发前，我们需要了解下面的问题：

- bk-cmdb是如何判断用户已经登录？
- bk-cmdb是如何跳转到登录系统？
- bk-cmdb是如何选择登录组件？ 
- bk-cmdb登录组件需要实现那些接口？

##### bk-cmdb 登录流程如下：

1. 先判断用户session在bk-cmdb是否已经存？
2. 已经存在，且session中的bk_token和cookie相等，转发请求。其他情况执行下一步。
1. 根据配置选择对应登录系统
2. 调用登录系统判断用户是否登录？
3. 如果用户已经登录，转发请求。否则执行下一步
4. 如果没有登录，返回登录系统跳转的地址。

#### 配置文件讲解，

登录系统的配置都在web.conf

开源版本在cmdb_adminserver/configures

文件内容解释：（v3.8.x分支，只写出用到参数）

- site bk-cmdb网站相关的配置

    site.domain_url bk-cmdb的域名，用来做跳转使用

- login 登录系统相关的配置
   
   1. login.version 当前系统系统使用的登录方式，现在支持两种blueking,skip-login,
 默认是blueking，蓝鲸登录，skip-login 表示没有登录系统，所有请求都是admin账户
 
   2. login配置下其他子项，都是登录系统用到其他的配置，可以根据登录方式组件的需要来配置，
   建议有检查是否登录，登录系统跳转的url等相关配置

示例:
```ini

[site]
domain_url = http://127.0.0.1:80

[login]
version=skip-login  
check_url = http://127.0.0.1/login/accounts/get_user/?bk_token=
bk_login_url = http://127.0.0.1/login/?app_id=%s&c_url=%s


```



#### 开发登录插件

需要实现功能：

interface 定义的位置：src/web_server/middleware/user/plugins/manager/manager.go

```interface

	LoginUser(c *gin.Context, config map[string]string, isMultiOwner bool) (user *LoginUserInfo, loginSucc bool)
	GetUserList(c *gin.Context, config map[string]string, params map[string]string) ([]*LoginSystemUserInfo, error)
	GetLoginUrl(c *gin.Context, config map[string]string, input *LogoutRequestParams) string



```

######  LoginUser 判断用户是否登录

参数：
 - c  gin 框架请求相关信息，用来获取request header和cookie信息。 
 - config  web.conf 配置文件内容， eg: map[string]string{"sit.domain_url":"xx", "login.version":"xxxx"}
 - isMultiOwner 保留项目，暂时未启用
 
 返回值：
 - user 返回当前登录用户的信息
 
```golang
 
type LoginUserInfo struct {
	UserName      string                      `json:"username"` // 用户的英文名，
	ChName        string                      `json:"chname"`   // 用户的中文名
	Phone         string                      `json:"phone"`    // 电话号码
	Email         string                      `json:"email"`    // 用户的邮箱 
	Role          string                      `json:"-"`        // 已废弃字段
	BkToken       string                      `json:"bk_token"` // bk_token 用来写入session
	OnwerUin      string                      `json:"current_supplier"` //预留字段
	OwnerUinArr   []LoginUserInfoOwnerUinList `json:"supplier_list"` //预留字段
	IsOwner       bool                        `json:"-"`             // 预留字段
	Extra         map[string]interface{}      `json:"extra"`         // 预留字段
	Language      string                      `json:"-"`             // 用户在在登录系统中的语言，zh-cn,en. zh-cn中文，en英文，默认值zh-cn
	AvatarUrl     string                      `json:"avatar_url"` //用户头像
	SupplierID    int64                       `json:"supplier_id"` // 预留字段，默认值即可
	MultiSupplier bool                        `json:"multi_supplier"` // 预留字段，默认值即可
}


```
 - loginSucc 是否登录成功
 
 其他：
 
 - 登录系统无法在当前bk-cmdb域名的cookie写入bk_token的的时候，可以在这里完成。 



######  GetUserList 从登录系统获取用户列表

参数：
 - c  gin 框架请求相关信息，用来获取request header和cookie信息。 
 - config  web.conf 配置文件内容， eg: map[string]string{"sit.domain_url":"xx", "login.version":"xxxx"}
 
 返回值:
 
- []*LoginSystemUserInfo 

```golang  

type LoginSystemUserInfo struct {
	CnName string `json:"chinese_name"` ### 中文名
	EnName string `json:"english_name"`  ### 英文名
}

```

- error 错信息


######  GetLoginUrl 获取登录系统的地址

参数：
 - c  gin 框架请求相关信息，用来获取request header和cookie信息。 
 - config  web.conf 配置文件内容， eg: map[string]string{"sit.domain_url":"xx", "login.version":"xxxx"}
 - input 使用的协议， https, http
 
返回值:

string 跳转到登录系统的地址，

建议：
登录系统都支持登录成功后跳转回原有系统，这个要设置好。 


##### 注册登录插件

```glang
package blueking

import (
	"configcenter/src/web_server/middleware/user/plugins/manager"
)

func init() {
	plugin := &metadata.LoginPluginInfo{
		Name:       "my bk-cmdb login system", //登录组件的描述
		Version:    "my-login",  // 登录组件的名字
		HandleFunc: &user{},  //提供服务集合, 提供LoginUser，GetUserList，GetLoginUrl功能。
	}
	manager.RegisterPlugin(plugin) //想插件管理注册插件
}

```


#####  提供服务

去 src/web_server/middleware/user/plugins/register 目录中新加一个文件。导入注册登录插件所在的package

可以参看blueking的方式。 

```golang 
package manager

import (
	_ "configcenter/src/web_server/middleware/user/plugins/method/blueking"
)

```


#####  查看效果

- 重新编译webserver服务
- 修改web.conf的login.version配置项的值等于注册登录插件中的plugin.Version
- 重启cmdb_adminserver使配置生效
- 重启cmdb_webserver查看效果
