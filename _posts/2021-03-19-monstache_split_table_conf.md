---
layout: post
title:  解决monstache在启动的时候无法将mongodb分表的同步到Elasticsearch
category: [monstache]
tags: [monstache]
---

###  背景

目前所在做的项目是做资产管理， 随着变化数据越来越多， 单一类型的资产数据无法在同一张表存储，后需经过使用多个表， 
之前为了实现全局的全文搜索， 是通过monstache将mongodb 中资产的实例的数据同步到Elasticsearch。


分表后在使用monstache 出现两个：
- 不支持按照mongodb collection 名字前缀进行数据
- monstache mapping 建立Elasticsearch 索引的时候不支持按collection 名字前缀进行聚合


### 如何解决问题？

#### 解决 monstache 无法按照 mongodb collection 名字前缀进行数据的问题？ 


monstache 在启动的时候是支持按正则来过滤不同步的collection 名字的配置项目（direct-read-dynamic-exclude-regex ）， 

- 方法1：使用正则中的非来完成按照前缀的同步。 例如： "?!prefix" 正则表示所有非pregfix 前缀名字的collection都不同步。 
但是golang regex panic 不支持。 
- 方法2： 将所有不需要同步的表名字全写direct-read-dynamic-exclude-regex， 耦合太紧，后需需要新加表都需要改配置


最终方案： 
在和monstache 作者沟通后，作者也认为上面的解决方案不可取，可以新建相应的功能。最后新加一个配置 direct-read-dynamic-include-regex
用来配置再启动的时候，按照配置的正则将符合条件名字的collection 中的数据同步到Elasticsearch中。
已提交PR， 并且合并[https://github.com/rwynn/monstache/pull/491](https://github.com/rwynn/monstache/pull/491). 
欢迎大家使用

#### 解决 monstache 无法按照 mongodb collection 名字前缀进行数据的问题？ 

monstache 在将mongodb collection中同步到Elasticsearch中的时候， 默认情况下会使用mongodb collection namespace 作为索引名字。 
也可以通过配置mapping 来做 mongodb collection namespace 到指定Elasticsearch索引中， 但是mapping 不支持按照正则或者前缀的方式。 

但是我们可以通过插件的方式来实现。 目前monstache 支持js和golang 开发的插件
为了降低部署成本。我这边使用的是js 的插件。直接在配置文件中写即可：


```angular2html

enable-patches = true

[[script]]
# namespace 限定js 插件处理的范围， 不填写的时候，是所有的表
#namespace = ""

routing = true 
script = """

var re = new RegExp("prefix")
module.exports = function(doc, ns, updateDesc) {

    // 找到满足条件的表
    if(re.test(ns)) {
        // _meta_monstache key是 monstache 自己描述字段， 其中的index 表示这个doc数据写到Elasticsearch用的索引
        doc["_meta_monstache"] = {"index":"es_index"};
    }
    
    return  doc 
}
"""


```