# Perfect New Relic Library for Linux

本项目为New Relic云监控服务的Swift 版本 Agent SDK.

本项目采用SPM软件包管理器编译，是[Perfect](https://github.com/PerfectlySoft/Perfect) 项目的一部分，但也可以作为独立模块使用。

## 发行说明

本项目只兼容 Ubuntu 16.04 和 Swift 4.0 工具链。

## 快速上手

请用[Perfect 软件助手](http://www.perfect.org/en/assistant/)编译本项目，否则请使用下列命令进行安装：

```
$ git clone https://github.com/PerfectlySoft/Perfect-NewRelic-linux.git
$ cd Perfect-libNewRelic-linux
$ sudo ./install.sh
```

在安装过程中，该产品会询问您在New Relic上注册的 **license key 许可证代码**, **application name 应用程序名称**, 开发所用的语言及其版本号。之后安装程序会将命令行 `newrelic-collector-client-daemon` 作为系统服务进行安装，安装完成后详细配置可以在这里找到： `/usr/local/etc/newrelic.service`.

请配置您工程的 Package.swift 文件并追加下列内容：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-NewRelic-linux.git", majorVersion: 3)
```

请将函数库导入您的程序（ ⚠️**注意**⚠️ 由于 Swift 4.0 linux 版本存在一个明显的编译器问题，因此 `PerfectNewRelic` 导入时必须配合 `Foundation` 函数库):

``` swift
import PerfectNewRelic
import Foundation
```

除了 Swift 与 C 一些语法上的转化差异之外，详细的编程手册情参考 [New Relic Agent SDK](https://docs.newrelic.com/docs/agents/agent-sdk/using-agent-sdk/using-agent-sdk)

## 配置

安装后的配置可以在 [New Relic - Configuring the Agent SDK](https://docs.newrelic.com/docs/agents/agent-sdk/installation-configuration/configuring-agent-sdk) 找到，请注意强烈推荐将其配置为**Daemon Mode 服务程序模式** 因为截至目前为止 *Embedded-mode嵌入模式* 仍然处于试验状态。

如果配置成功，您可以很轻易地创建NewRelic类的例程：

``` swift
let nr = try NewRelic()
```

- 还可以创建回调函数来判读后台服务的状态：

``` swift
nr.registerStatus { code in
	guard let status = NewRelic.Status(rawValue: code) else {
		// 出错了
	}//end guard
	switch status {
		case .STARTING: // NewRelic Daemon 服务正在启动
		case .STARTED: // NewRelic Daemon 服务启动成功
		case .STOPPING: // NewRelic Daemon 服务正在停止
		default: // NewRelic Daemon 服务已经关闭
   }//end case
}//end callback
```

## Agent SDK 配置和限额

根据 [New Relic Agent SDK 配置说明](https://docs.newrelic.com/docs/agents/agent-sdk/installation-configuration/limiting-or-disabling-agent-sdk-settings)，以下配置信息同样适用于Perfect NewRelic函数库：

目标操作…… | 配置方案……
-------------------|---------------------
在事务操作过程中关闭服务器性能数据采集|`nr.enableInstrumentation(false)`
在事物操作过程中配置允许用于追踪的最大分段数量|`let t = try Transaction(nr, maxTraceSegments: 50)` // 最多猜忌50个追踪段

## API 速查手册

根据 [New Relic 开发工具使用说明](https://docs.newrelic.com/docs/agents/agent-sdk/using-agent-sdk/using-agent-sdk)，Perfect NewRelic 函数库提供与其C语言函数库完全一样的功能：

### 性能指标和监控

函数 | `recordMetric()`
---|---
举例|`try nr.recordMetric(name: "ActiveUsers", value: 25)`
描述|记录一个自定义指标
参数| - name: 指标名称； <br> - value: 指标数值；

函数 | `recordCPU()`
---|---
举例|`try nr.recordCPU(timeSeconds: 5.0, usagePercent: 1.2)`
描述|记录CPU用户占用时间（秒）以及CPU占用百分比。
参数| - timeSeconds: Double, 用户CPU消耗时间 <br> - usagePercent: Double, CPU 占用百分比

函数 | `recordMemory()`
---|---
举例|`try nr.recordMemory(megabytes: 32)`
描述|记录当前内存用量
参数| - megabytes: Double，内存消耗（单位：兆字节）

### Transaction 事务对象

Perfect NewRelic 的事物对象Transaction 被定义为一个Swift类，构造函数如下：

``` swift
public init(_ instance: NewRelic,
    webType: Bool? = nil,
    category: String? = nil,
    name: String? = nil,
    url: String? = nil,
    attributes: [String: String],
    maxTraceSegments: Int? = nil
  ) throws
```

#### 参数说明

- instance: NewRelic 实例, **必填参数**.
- webType: **可选参数**，真表示 WebTransaction，假则为其他类型。默认为真。
- category: **可选参数**. 事务分类名称，默认为 'Uri'
- name: **可选参数**. 事务名称
- url: **可选参数**. 对于 WebTransaction 所使用的URL链接
- attributes: **可选参数**. 事务的属性，字典类型，字典内每个字段对应一个变量值。
- maxTraceSegments: **可选参数**. 在同一个事务内可以跟踪的最大区段数量限额。默认最大值为2000，也就是说在同一个事务里，如果区段数量超过计划限额（四倍的apdex_t）时，只有头2000个分段将用于跟踪报告。

#### 事务类初始化代码示范

``` swift
let nr = NewRelic()
let t = try Transaction(nr, webType: false,
	category: "my-class-1", name: "my-transaction-name",
	url: "http://localhost",
	attributes: ["tom": "jerry", "pros":"cons", "muddy":"puddels"],
	maxTraceSegments: 2000)
```

#### 错误通知

Perfect NewRelic 为事务提供 `setErrorNotice()` 函数：

``` swift
try t.setErrorNotice(
	exceptionType: "my-panic-type-1",
	errorMessage: "my-notice",
	stackTrace: "my-stack",
	stackFrameDelimiter: "<frame>")
```

`setErrorNotice()` 方法参数说明：

- exceptionType: 出错类型
- errorMessage: 错误信息
- stackTrace: 堆栈跟踪
- stackFrameDelimiter:  堆栈分隔符

#### Segments 区段

一个事务类型中的区段类型可以是 Generic （通用）、DataStore（数据存储）或者External（外部调用），参考如下代码

``` swift
// 假如 t 是一个Transaction 对象例程
let root = try t.segBeginGeneric(name: "my-segment")
// 执行某些常规操作
try t.segEnd(root)

// 注意：下面调用方法采用自动 Obfuscation 混合方法并采用自动跟踪回滚：
let sub = try t.segBeginDataStore(table: "my-table", operation: .INSERT, sql: "INSERT INTO table(field) value('000-000-0000')")
// 执行某些数据操作
try t.segEnd(sub)

let s2 = try t.segBeginExternal(host: "perfect.org", name: "my-seg")
// 执行某些外部调用
try t.segEnd(s2)
```

参数说明:

- parentSegmentId: 父区段代码。默认为零，也就是` NewRelic.ROOT_SEGMENT`.
- name: 区段名称

