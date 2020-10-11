---
layout: post
title:  蓝鲸配置平台-蓝鲸cmdb接入层支持对第三方组件访问的转发开发
category: [bk-cmdb]
tags: [bk-cmdb]
---


#### 前提

蓝鲸配置平台（bk-CMDB）是一个面向资产及应用的企业级配置管理平台。
是蓝鲸运维体系一部分，蓝鲸社区版本是免费，社区地址[https://bk.tencent.com/]()
并且蓝鲸很多组件已经开源，蓝鲸cmdb就是其中之一，

蓝鲸配置平台接入层API server目前仅支持对场景层开发，无法对第三方开发的场景层做代理转发。 



#### 目标

- 实现项目模块
- 降低模块接入成本
- 完善模块文档


#### 需要做的工作

- 服务的动态发现
- 根据规则转发请求 
- auth 设计（暂不支持）


#### 接入层（apiserver） 对第三方组件访问的转发开发实例

代码放到 src/apiserver/service/match/目录中即可


###### 使用服务发现发现服务

```
	RegisterService(服务名字, 实现URL匹配方法)
```

###### 实现URL匹配方法


```
func MatchMigrate(req *restful.Request) (from, to string, isHit bool) 
```




eg: 

``` golang
package match
import (
	"strings"
	"configcenter/src/common/types"
	"github.com/emicklei/go-restful"
)
func init() {
	RegisterService(types.CC_MODULE_MIGRATE, MatchMigrate)
}
func MatchMigrate(req *restful.Request) (from, to string, isHit bool) {
	uri := req.Request.RequestURI
	from, to = RootPath, "/migrate/v3"
	switch {
		
	case strings.HasPrefix(uri, RootPath+"/authcenter/init"):
	// 将请求URL未/api/v3/authcenter/init，转发到当前服务
		isHit = true
	case strings.HasPrefix(uri, RootPath+"/find/system/config_admin"):
		// 将请求URL未/api/v3/find/system/config_admin，转发到当前服务
		isHit = true
	default:
		isHit = false
	}
	return from, to, isHit
}
```
