# 远程日志

使用 `PerfectLogger` 模块，本地服务器应用产生的事件可以注册并记录到远程服务器上，也可以用于在控制面板上查看。

生产级的日志服务器 [Perfect Log Server](https://github.com/PerfectServers/Perfect-LogServer) 是单独的开源项目，可以在您的服务器上进行部署。


## 使用方法

首先请在您的项目中Pacakge.swift 文件增加依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Logger.git", majorVersion: 3),
```

下一步是在程序开始前导入日志记录模块：

``` swift 
import PerfectLogger
```

## 配置

使用远程日志功能需要配置三个参数：

``` swift
// 钥匙代码
RemoteLogger.token = "<your token>"

// 应用程序序号（可选项）App ID
RemoteLogger.appid = "<your appid>"

// URL 地址，即远程目标具有日志记录功能的服务器地址
// 请注意，这里没用完整的API路径，仅仅是服务器域名和端口。
RemoteLogger.logServer = "http://localhost:8181"

```

## 编程方法

如果要将本地事件记录到远程服务器：

``` swift
var obj = [String: Any]()
obj["one"] = "donkey"
RemoteLogger.critical(obj)
```

## 关联事件关系：使用“eventid”事件编号

每一条事件都有一个唯一的事件编号。如果在记录事件的指令中附带了另一个事件的编号，则能将当前正在记录的事件与附加事件进行关联：

``` swift
let eid = RemoteLogger.critical(obj)
RemoteLogger.info(obj, eventid: eid)
```

返回的事件编码在程序中明确的标记为 `@discardableResult` 因此您可以忽略这些变量的内存管理，不会对系统造成垃圾堆积。
