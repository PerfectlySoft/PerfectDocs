# Google Analytics Measurement Protocol

谷歌网站分析协议函数库相当于网页中的谷歌分析工具在服务器端的实现，也就是说，您可以用日志记录任何一种网络活动——无论是原始的TCP/UDP事件，还是某种被触发的特定事件，比如AJAX。

## 编程手册

详细编程手册请参考这里： [https://www.perfect.org/docs/api-Perfect-GoogleAnalytics-MeasurementProtocol.html](https://www.perfect.org/docs/api-Perfect-GoogleAnalytics-MeasurementProtocol.html).

该手册详细说明了函数库内的所有接口、方法和属性。

## 配置方法

`PerfectGAMeasurementProtocol`结构用于在应用程序运行空间内设置属性编号和点击类型：

``` swift
PerfectGAMeasurementProtocol.propertyid = "UA-XXXXXXXX-X"
PerfectGAMeasurementProtocol.hitType = "pageview"
```

## 编译

请在您项目的Package.swift文件中增加如下依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-GoogleAnalytics-MeasurementProtocol.git", majorVersion: 3)
```

## 使用范例

设置执行和登记某个事件的方法：

```swift
PerfectGAMeasurementProtocol.propertyid = "UA-XXXXXXXX-X"
let gaex = PerfectGAEvent()
gaex.user.uid = "donkey"
gaex.user.cid = "kong"
gaex.session.ua = "aua"
gaex.traffic.ci = "ci"
gaex.system.fl = "x"
gaex.hit.ni = 2


do {
	let str = try gaex.generate()
	print(str)
	let resp = gaex.makeRequest(useragent: "TestingAPI1.0", body: str)
	print(resp)
} catch {
	print("\(error)")
}

```

常见的点击类型和配置请参考下列文档：
[https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#commonhits](https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide#commonhits) 

