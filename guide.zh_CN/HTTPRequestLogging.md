# HTTP 请求的日志记录

如果希望把HTTP请求写入文件，请使用`Perfect-RequestLogger` 函数库

## 使用方法

请在您的项目文件 `Package.swift` 中增加以下依存关系：

```swift
.Package(url: "https://github.com/dabfleming/Perfect-RequestLogger.git", majorVersion: 0, minor: 3)
```

对于要调用该功能的源程序，请在源程序文件开头增加导入语句：

``` swift 
import PerfectRequestLogger
```

这样在您的主程序 `main.swift`初始化服务器对象`server`之后，就可以将请求写入文件了：

```swift
// 初始化一个日志记录器
let myLogger = RequestLogger()

// 增加过滤器
// 首先增加高优先级的过滤器
server.setRequestFilters([(myLogger, .high)])
// 最后增加低优先级的过滤器
server.setResponseFilters([(myLogger, .low)])
```

这些请求/响应过滤器能够在调用HTTP请求和响应的闭包之前实现功能挂钩。


## 设置日志文件的存储位置

默认的服务器日志文件路径为`/var/log/perfectLog.log`。您可以通过在主程序`main.swift`中设置全局变量`requestLogFileLocation` 来改变这个位置：

``` swift
requestLogFileLocation = "/var/log/myLog.log"
```

## 输出效果：

```
[INFO] [servername/WuAyNIIU-1] 2016-10-07 21:49:04 +0000 "GET /one HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000436007976531982s
[INFO] [servername/WuAyNIIU-2] 2016-10-07 21:49:06 +0000 "GET /two HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000207006931304932s
[INFO] [servername/WuAyNIIU-3] 2016-10-07 21:49:07 +0000 "GET /three HTTP/1.1" from 127.0.0.1 - 200 66B in 0.00014495849609375s
```

该模块是在大卫·弗莱明的工作基础之上完成：[David Fleming](https://github.com/dabfleming).
