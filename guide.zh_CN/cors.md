## CORS （跨来源资源共享）安全功能

跨来源资源共享 (CORS) 是开放互联网的重要组成部分。换言之，没有CORS的支持，任何互联网框架都不会是完整的。

参考蒙梭·侯赛因的[html5rocks.com introduces CORS very effectively](https://www.html5rocks.com/en/tutorials/cors/)教学内容：

> 互联网服务接口API就好像编织这张大网用的丝和线一样，但是使用过程中资源跨域传输可能会有些困难，特别是这些跨域请求受限于如 JSON-P 或者设置代理服务器等技术 —— JSON-P有安全隐患，而代理服务器的设置和维护会比较复杂。
> 
> 跨来源资源共享（CORS）是一个W3C标准，允许从浏览器进行跨域访问，在XMLHttpRequest基础上实现。该标准允许程序员处理资源时，采用与同区域请求相同的方式。
> 
> CORS的使用方法很简单。假设网站“张三.com”有一些数据需要“李四.com”获取，那么传统方式下受限于浏览器的“会话目标网站的唯一性限制”，这种方式是不可能被允许的。但是，如果有了CORS的支持，“张三.com”可以增加一些特殊的响应头数据，以允许“李四.com”进行访问。
> 
> 如本示范内容所示，CORS支持需要服务器和客户端之间协同。如果您是前端工程师则不需要理会这些细节。本文剩下的内容是展示客户机如何实现跨来源请求，并且服务器如何进行配置以支持CORS。

[Perfect 会话](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) 模块包括了CORS配置支持，使得您的服务函数接口API可以根据需要允许或者禁止此类资源共享。

如果您的工程软件配置文件 Package.Swift 包括了 Perfect Sessions 会话管理，或者任何带有数据源驱动的应用实现，则您的工程已经包括CORS支持；但是请注意，默认情况下该支持处于 **关闭** 状态。

## 相关例子

* [Perfect-Session-Memory-Demo](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)

## 配置

``` swift
// 总开关；默认为关闭。
SessionConfig.CORS.enabled = true

// 允许进行跨来源资源共享的主机清单。
// 如果希望不限制任何主机访问，则只需要保留一个通配符元素“*”即可。
SessionConfig.CORS.acceptableHostnames = ["*"]

// 否则如果需要追加特定域名
SessionConfig.CORS.acceptableHostnames.append("http://www.test-cors.org")

// 在域名中的开始和结束可以使用通配符
SessionConfig.CORS.acceptableHostnames.append("*.example.com")
SessionConfig.CORS.acceptableHostnames.append("http://www.domain.*")

// 允许使用的方法列表
public var methods: [HTTPMethod] = [.get, .post, .put]

// 允许的自定义头数据
public var customHeaders = [String]()

// Access-Control-Allow-Credentials 是否允许访问机密信息
// 默认情况下标准 CORS 请求不会发送或者设置任何cookies。
// 如果希望允许在跨域请求中使用cookies，则请将此处设置为真。
public var withCredentials = false

// 内容缓冲时限，单位时秒
// 默认为0，也就是关闭内容缓冲
public var maxAge = 3600

```

当一个 CORS 请求发到服务器后，如果选项配置不匹配任何允许的主机，则该配置会提醒浏览器不要接受这种资源。

如果服务器决定产生 CORS 头数据，则下列头数据会追加到响应内容：

``` swift
// 允许的 HTTP 方法。
// 由上述配置数组生成的内容：
Access-Control-Allow-Methods: GET, POST, PUT

// 如果来源是被认可的，则来源会被自动回应给请求者
// (即使配置为通配符 *)
Access-Control-Allow-Origin: http://www.test-cors.org

// 如果服务器允许cookies
Access-Control-Allow-Credentials: true

// 允许在服务器上设置缓冲（有效期为1个小时）
Access-Control-Max-Age: 3600
```

可以参考以下工具来验证 CORS 冠名和响应结果[http://www.test-cors.org](http://www.test-cors.org)