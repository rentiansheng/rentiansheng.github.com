---
layout: post
title: golang 通用指标框架之数据展示-代码约束
category: [golang]
tags: [golang,metric]
---

## 1. 任务定义
通过对任务属性抽象和集合， 将任务定义分为， meta信息， 任务周期信息， 插件信息，任务执行动态信息
- meta信息： 主要就是任务名字，状态
- 任务周期信息： 周期类型，周期计算方式，需要计算的周期数
- 插架信息： 任务需要使用到插件及插件的配置
- 任务执行动态信息： 任务开始周期，最后一次执行完成时间

[代码位置 task.go](https://github.com/rentiansheng/calculate_metric/blob/master/src/define/task.go)
```go
type MetricTask struct {
	// 任务的名字，同时也是指标名字
	TaskName string `json:"task_name"  gorm:"column:task_name"`
	// 指标周期（1年，2季，3月，4周，5日，6时）（接口必须）
	TaskCycle int8 `json:"task_cycle" gorm:"column:task_cycle"`
	// 周期执行方式，1 周期结束后执行，2周期中每天计算一次
	CycleMode      CycleModeType `json:"cycle_mode" gorm:"column:cycle_mode"`
	CalculateCycle uint8         `json:"calculate_cycle" gorm:"column:calculate_cycle"`
	// 任务状态， 1 正常，可以允许， 2. 暂停，不被执行 3. 待删除
	TaskStatus StatusEnumType `json:"task_status" gorm:"column:task_status"`

	// TaskStart 任务采集数据的开始时间， 有start+cycle 可以得到结束时间
	TaskStart   uint64                              `json:"task_start" gorm:"column:task_start"`
	Collect     MetricTaskPluginCollectConfig       `json:"collect" gorm:"column:collect"`
	Filters     MetricTaskPluginConfigArr           `json:"filters" gorm:"column:filters"`
	Aggregators MetricTaskPluginAggregatorConfigArr `json:"aggregators" gorm:"column:aggregators"`
	// Output 使用到output 插件
	Output      MetricTaskPluginOutputConfig        `json:"output" gorm:"column:output"`
	//Power          MetricPower                         `json:"power" gorm:"column:power"`
	// LastFinishTime 任务上一次完成时间 
	LastFinishTime uint64 `json:"last_finish_time" gorm:"column:last_finish_time"`

	// OutputIndexName 存储output 插件返回存储的信息
	OutputIndexName string `json:"output_index_name" gorm:"column:output_index_name"`
}


```

## 2. 插架之间数据交换约束

[代码位置 data.go](https://github.com/rentiansheng/calculate_metric/blob/master/src/define/data.go)

将任务执行分为数据处理和存储两个阶段。
- 数据处理：包含获取周期内数据，数据过滤，数据聚合
- 存储结果：将指标和符合指标数据持久化



### 2.1 数据处理约束-Record
用来约束collect,filter,aggregator之间数据交换。
   - Data() 是用来做数据对比的数据
   - Update() 用来提供对原始更新，设计上是留给filter来做数据处理
   - Field() 用来做统计的数据
   - UUID()  框架中用来做数据去重使用
```go
type Record interface {
	Data() map[string]string
	Update(key, value string) error
	Field() map[string]float64
	UUID() string
}

```

### 2.2 存储结果约束-MetricData
传给output 插件的数据，计算的结果数据
```go
// MetricData 计算产生的数据
type MetricData struct {
    MetricKey string
    Value     map[string]float64     `json:"value"`
    Extra     map[string]interface{} `json:"extra"`
}
```


## 3.  插架约束

[代码位置 define.go](https://github.com/rentiansheng/calculate_metric/blob/master/src/define/plugins.go)
实现自定义插件的时候，需要实现的代码约束 
```go
// Collect 收集数据插件
type Collect interface {
	// Name 插件的名字，必须全局唯一
	Name() string
	// Keys 用做计算的维度，可以任务是group by 中分组的名字
	Keys(ctx context.Context) ([]string, error)
	// Run 执行统计任务， 这个有框架控制并发执行的
	Run(ctx context.Context, key string, start, end uint64, input chan Record) error

	// SetConfig 修改配置
	SetConfig(ctx context.Context, config []byte) error
	// Description 用来描述配置
	Description() string

	// GetHooks 获取注入插件，用来放到流程不同流程执行
	//GetHooks() Hooks
}

// Filter data transform
type Filter interface {
	// Name 插件的名字，必须全局唯一
	Name() string
	// Run 执行
	Run(ctx context.Context, key string, data Record) error
	// SetConfig 初始化的时候，用来放入配置
	SetConfig(ctx context.Context, config []byte) error
	// Description 用来描述配置
	Description() string
}

// Aggregator calculate metric
type Aggregator interface {
	// Name 插件的名字，必须全局唯一
	Name() string
	// Run 执行
	Run(ctx context.Context, key string, data Record) error
	// SetConfig 初始化的时候，用来放入配置
	SetConfig(ctx context.Context, config []byte) error
	// Description 用来描述配置
	Description() string
	// Metric 存储的时候，获取当前插件计算的值
	Metric(ctx context.Context) (string, float64)
	MetricExtra(ctx context.Context) (string, interface{}, bool)
	SetMetricMetadata(ctx context.Context, data MetricMetadata) error
}

// Output record to storage
type Output interface {
	// Name 插件的名字，必须全局唯一
	Name() string
	// Write 输出数据到目标
	Write(ctx context.Context, data OutputData) error
	// Exists 判断key 在周期内是否已经执行过，任务分片后，判断是否需要跳过的依据
	Exists(ctx context.Context, metricKey string) (bool, error)
	// Description 用来描述配置
	Description() string
	// IndexName string
	IndexName(ctx context.Context) (string, error)
	SetMetricMetadata(ctx context.Context, data MetricMetadata) error
}
```




