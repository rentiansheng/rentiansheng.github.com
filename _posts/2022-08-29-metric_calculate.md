---
layout: post
title: golang 通用指标框架之数据展示
category: [golang]
tags: [golang,metric]
---

## 1. 背景
最近在做一个研发效能指标展示的项目，该项目主要的功能收集第三方系统数据，按照周期计算，展示图表。

系统主要业务逻辑在数据同步，周期数据聚合，数据计算，结果存储等几方面。

业务需求主要是在指标规则和数据范围频繁发生，变化？ 迫切需要标准化计算框架。
降低统计指标开发难度，提高统计指标任务开发效率，实现统计流程与业务逻辑解构及指标计算过程配置化和插件化。


## 2. 面临的问题
设计面临的问题：
![metric_calculate_question](/img/metrics/metric_calculate_question.jpg)

业务层面：

    任务执行失败，如何重试机制
    任务调度和负载
    任务并发执行协调
    任务分批执行
    公共逻辑没有复用，比如： 周期计算，数据统计，结果存储
    数据，业务。流程逻辑解耦
    指标结果存储优化。（分表，数据清理，历史记录）
    指标重复计算
    数据去重

## 3. 如何设计解决问题？

### 3.1 采用配置，流水线及插件定义完成指标计算过程 
![metric_plugins_config](/img/metrics/metric_plugins_config.png)

### 3.2 框架设计
在开发角度一个指标任务执行流程
![metric_calculate_arch](/img/metrics/metric_arch.png)
在机器的角度
![metric_framework_workers](/img/metrics/metric_framework_workers.png)

用户参与功能及框架功能角度
![metric_framework_function](/img/metrics/metric_framework_function.png)


#### 3.2.1 用户参与的功能
用户参与的功能。主要是在plugins， 插件也是可以公用的.
数据在按照collect → filter → aggregate → filter → output   单方向顺序执行
同类型的多个插件按照顺序执行

##### 3.2.1.1 数据收集(collect)
用来获取周期内需要统计的数据，数据来源不限，可以是mysql，es，第三方服务。

##### 3.2.1.1 数据处理(filter)
用来实现统计数据进行转换和过滤，比如： 对已有字段拆分，合并，去重等。

##### 3.2.1.1 统计数据(aggregator)
按照条件进行数据统计， 比如 计数(count), 平均(avg), 最小值(min),最大值(min)

##### 3.2.1.1 数据处理(filter)
用来实现统计结果进行转换， 比如： 对已有字段拆分，合并等。

##### 3.2.1.1 结果(output)
将统计结果持久化。比如：mysql

#### 3.2.2 框架功能
    任务分片
    任务重试
    并发处理
    任务协调
    数据去重
    周期管理
    结果分表

## 4 代码及约束 未完成







