---
layout: post
title:   telegraf agent 源码分析
category: [telegraf]
tags: [telegraf]
---

# 1 telegraf 

telegraf 是开源agent服务，可收集系统和服务的metric数据. 
telegraf 从数据的collecting, processing, aggregating, and writing 都采用插件的设计，非常灵活，易扩展。
可以通过官方和自定义插件完成业务功能。

有四种不同类型的插件：
- Input Plugins collect metrics from the system, services, or 3rd party APIs
- Processor Plugins transform, decorate, and/or filter metrics
- Aggregator Plugins create aggregate metrics (e.g. mean, min, max, quantiles, etc.)
- Output Plugins write metrics to various destinations


telegraf 设计是非常优秀，通过插件系统，将业务逻辑，数据差异放到不同流程的插件中处理，流程之间通过channel进行数据交互，解耦非常彻底。
agent 只需要完成插件管理，数据流向，插件调度。

# 2 telegraf agent

## 2.1 agent 如何管理数据？
telegraf/agent/agent.go  做为agent 功能核心代码。 
主要通过 Agent,inputUnit,processorUnit,aggregatorUnit,outputUnit结构体来完成agent 的功能。
建议看源码，注释写非常明确。
- agent 用来管理agent 配置，
- inputUnit 用来管理Input 插件
- processorUnit 用来管理Processor 插件
- aggregatorUnit 用来管理Aggregator 插件
- outputUnit 用来管理Output 插件

## 2.2 agent 如何约束插件？

### 2.2.1 agent 插件全局约束
全局约束是所有类型插件都适用的。

#### 2.2.1.1 必须约束如下
- PluginDescriber 用来描述插件，SampleConfig()在./telegraf --sample-config 可以看到
```go

// PluginDescriber 插件描述
type PluginDescriber interface {
	// SampleConfig returns the default configuration of the Processor
	SampleConfig() string

	// Description returns a one-sentence description on the Processor
	Description() string
}
```

#### 2.2.1.2 可选约束
- Initializer 初始化插件的操作

```go
// Initializer 插件初始化约束
type Initializer interface {
	// Init performs one time setup of the plugin and returns an error if the
	// configuration is invalid.
	Init() error
}

```

### 2.3.1 agent 插件约束

实现自定义插件的时候，需要根据插件类型的约束做实现， 
实现后还需要注册插件， 需要配合导入包的方式。具体可以看 telegraf/{inputs/outputs/processors/aggregators}/all/aal.go
#### 2.3.1.1 Input 插件
telegraf/input.go
```go
type Input interface {
	PluginDescriber

	// Gather takes in an accumulator and adds the metrics that the Input
	// gathers. This is called every agent.interval
	// 实现数据的收集功能，metric数据输入
	Gather(Accumulator) error
}

```

#### 2.3.1.2 Processor 插件
telegraf/processor.go
```go
// Processor is a processor plugin interface for defining new inline processors.
// these are extremely efficient and should be used over StreamingProcessor if
// you do not need asynchronous metric writes.
type Processor interface {
	PluginDescriber

	// Apply the filter to the given metric.
	Apply(in ...Metric) []Metric
}

```

#### 2.3.1.3 aggregator 插件
telegraf/processor.go
```go
type Aggregator interface {
	PluginDescriber

	// Add the metric to the aggregator.
	Add(in Metric)

	// Push pushes the current aggregates to the accumulator.
	Push(acc Accumulator)

	// Reset resets the aggregators caches and aggregates.
	Reset()
}

```

#### 2.3.1.4 Output 插件
telegraf/output.go
```go
type Output interface {
	PluginDescriber

	// Connect to the Output; connect is only called once when the plugin starts
	Connect() error
	// Close any connections to the Output. Close is called once when the output
	// is shutting down. Close will not be called until all writes have finished,
	// and Write() will not be called once Close() has been, so locking is not
	// necessary.
	Close() error
	// Write takes in group of points to be written to the Output
	Write(metrics []Metric) error
}

```

## 2.3 agent 代码

telegraf/agent/agent.go 关于agent 的代码，需要注意插件启动顺序，插件的管理，采集异常处理。
### 2.3.1 agent Run 

```go
// Run starts and runs the Agent until the context is done.
func (a *Agent) Run(ctx context.Context) error {
    // 执行插件的初始化，前提是插件实现telegraf.Initializer
	a.initPlugins()
    
    // 启动ouput，这里主要是执行（*output).connectOutput 方法，
    // 设计来源类似linux将一切皆文件的思想，connectOutput类似的open。
	next, ou, err := a.startOutputs(ctx, a.Config.Outputs)

	var apu []*processorUnit
	var au *aggregatorUnit
	if len(a.Config.Aggregators) != 0 {
		aggC := next
		if len(a.Config.AggProcessors) != 0 {
			aggC, apu, err = a.startProcessors(next, a.Config.AggProcessors)
		}
		next, au = a.startAggregators(aggC, next, a.Config.Aggregators)
	}

	var pu []*processorUnit
	if len(a.Config.Processors) != 0 {
		next, pu, err = a.startProcessors(next, a.Config.Processors)
	}

	iu, err := a.startInputs(next, a.Config.Inputs)
	
	a.runOutputs(ou)
	a.runProcessors(apu)
	a.runAggregators(startTime, au)

	a.runProcessors(pu)
	a.runInputs(ctx, startTime, iu)
	
}


```





