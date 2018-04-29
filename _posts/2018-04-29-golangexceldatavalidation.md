---
layout: post
title: Golang EXCEL 列，单元格数据验证(下拉列表，数字文本长度校验)
category: [golang]
tags: [golang, excel, 数据验证]
---






## 1. 前提
---

最近Golang开发的项目中用到excel 导入，导出。数据导出没什么问题， 直接写excel 就行了。但是导入模版遇到一些问题。

问题如下：
   
    1. db中数据定义为字符串，字段如果用户填写都是数字，获取数据的时候，该字段就是数字，需要后端转换
	2. 枚举类型字段，填写难度过大。用户需要知道对应的值
	3. 字段范围值，填写比较困难，（如用户，类型等）
	3. 无法限定字段填写内容的长度
	4. 无法限定数字填写范围

为什么不直接放一个excel文件， 项目中数据导入，导出字段是不固定，需要根据个人的配置实字段来导入，导出

## 2. Golang EXCEL 类库选择
---

经过上网查找发现两个现在使用比较多golang 操作 excel 类库
1. [excelize](https://github.com/360EntSecGroup-Skylar/excelize)
2. [xlsx](https://github.com/tealeg/xlsx)

最总，选择了xlsx, xlsx在star 和fork 比 excelle多。但是excelize最近活跃度上更活跃，文档更详细（有中文）


## 3. 使用EXCEL 数据验证
---
xlsx在使用中，基本功能没什么问题，但是在做高级功能的时候，发现很多高级功能没有时间，有的实现过于粗暴。 数据校验功能根本没有实现。本来想去copy excelize的代码，结果，发现他们也没有实现， 还在todo列表中躺着。无奈只能自己实现。

实现功能代码在：
[支持数据验证](https://github.com/rentiansheng/xlsx) 代码已经提交PR,但是未合并。
现在简单粗暴的实现 excel 数据校验中的list，rang 验证（时间，日期 需要转换成数字，现在未实现）已经确认可用数据校验有 list（展示dorp box ），rang(数字，小数，文本长度)， 用户输入的内容不对会有提示。 

关于数据校验的代码的测试例子在[数据验证test代码](https://github.com/rentiansheng/xlsx/blob/dev_master/datavalidation_test.go)
可以go test 查看生产excel内容。 

## 4. 开发EXCEL 需要准备的
---

excel 是openxml 来描述的一个zip压缩文件。
office中的文件都是类似的。 微软已经将文档公开。

工具：
open xml sdk 2.0 的工具，可以查看excel 文件的openxml代码，提供校验，文档，生成代码功能（不是golang）




