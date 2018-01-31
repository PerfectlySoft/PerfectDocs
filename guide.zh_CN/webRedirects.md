# Web 重定向

Perfect WebRedirects 模块能够过滤特定路由（包括通配符路由）并按照匹配结果执行请求重定向服务。

这一点对于搜索引擎优化而言可能是非常重要的。比如，如果之前有一个静态文件`/about.html`被更新为新的路径`/about`，那么对不起您的网站之前辛辛苦苦积累的搜索引擎排名就全丢了。

关于本功能模块的详细用法和展示参考项目可以在这里找到：[Perfect-WebRedirects-Demo](https://github.com/PerfectExamples/Perfect-WebRedirects-Demo).

## 使用方法

首先在您的项目中Package.swift文件增加依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-WebRedirects", majorVersion: 3),
```

然后在您Perfect服务器的路由过滤器配置程序中，比如 `main.swift` 里增加如下过滤器：

``` swift
import PerfectWebRedirects
```

之后就可以追加过滤器了：

``` swift
// 在config环节中增加过滤器：
[
	"type":"request",
	"priority":"high",
	"name":WebRedirectsFilter.filterAPIRequest,
]
```

如果您还需要追加请求日志处理器，而且重定向过滤器排在请求处理器的后面，则二者都会被触发响应，新的请求不但会首先记录，而且会被正确的重定向。

## 配置文件

路由配置文件是一个JSON文件，形式如下：`/config/redirect-rules/*.json` 

```
{

  "/test/no": {
	"code": 302,
	"destination": "/test/yes"
  },

	"/test/no301": {
		"code": 301,
		"destination": "/test/yes"
  },
  
	"/test/wild/*": {
		"code": 302,
		"destination": "/test/wildyes"
  },

	"/test/wilder/*": {
		"code": 302,
		"destination": "/test/wilding/*"
  }

}
```

注意该目录下可以存放多个JSON文件；所有该目录下的JSON文件都会在过滤器首次触发时加载。

其中，每个过滤器的关键词是原有路径，而值对应的是目标需要重定向的新路径。