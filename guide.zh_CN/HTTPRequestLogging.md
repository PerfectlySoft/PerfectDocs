# HTTP 请求的日志记录

如果希望把HTTP请求写入日志文件，请使用`Perfect-RequestLogger` 函数库

## 相关范例

* [Perfect-HTTPRequestLogging](https://github.com/PerfectExamples/Perfect-HTTPRequestLogging)
* [Perfect-Session-Memory-Demo](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)

## 使用方法

请在您的项目文件 `Package.swift` 中增加以下依存关系：

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-RequestLogger.git", majorVersion: 3)
```

对于要调用该功能的源程序，请在源程序文件开头增加导入语句：

``` swift 
import PerfectRequestLogger
```

## 对于 PerfectHTTP 2.1 以上版本

在您服务器 `main.swift` 增加下列内容

```swift
// 初始化日志记录器
let httplogger = RequestLogger()

// 配置服务器
var confData: [String:[[String:Any]]] = [
	"servers": [
		[
			"name":"localhost",
			"port":8181,
			"routes":[],
			"filters":[
				[
					"type":"response",
					"priority":"high",
					"name":PerfectHTTPServer.HTTPFilter.contentCompression,
					],
				[
					"type":"request",
					"priority":"high",
					"name":RequestLogger.filterAPIRequest,
					],
				[
					"type":"response",
					"priority":"low",
					"name":RequestLogger.filterAPIResponse,
					]
			]
		]
	]
]
```
其中配置关键在于增加下列过滤器：

``` swift
[
	"type":"request",
	"priority":"high",
	"name":RequestLogger.filterAPIRequest,
],
[
	"type":"response",
	"priority":"low",
	"name":RequestLogger.filterAPIResponse,
]
```
这些请求/响应过滤器能够用于触发HTTP访问时记录日志。



## 如果使用 PerfectHTTP 2.0 版本服务器

在您服务器 `main.swift` 增加下列内容

```swift
// 初始化日志记录器
let httplogger = RequestLogger()

// 增加过滤器
// 请求过滤器，高优先触发
server.setRequestFilters([(httplogger, .high)])
// 响应过滤器，最后一个触发
server.setResponseFilters([(httplogger, .low)])
```

这些请求/响应过滤器能够用于触发HTTP访问时记录日志。

## 设置日志文件的存储位置

默认的服务器日志文件路径为`/var/log/perfectLog.log`。您可以通过在主程序`main.swift`中设置属性 `RequestLogFile.location`:

``` swift
RequestLogFile.location = "/var/log/myLog.log"
```

## 输出效果：

```
[INFO] [62f940aa-f204-43ed-9934-166896eda21c] [servername/WuAyNIIU-1] 2016-10-07 21:49:04 +0000 "GET /one HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000436007976531982s
[INFO] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [servername/WuAyNIIU-2] 2016-10-07 21:49:06 +0000 "GET /two HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000207006931304932s
```

该模块是在大卫·弗莱明的工作基础之上完成：[David Fleming](https://github.com/dabfleming).
