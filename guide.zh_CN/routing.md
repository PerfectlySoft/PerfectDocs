# HTTP路由

HTTP请求/响应路由是用于决定在当前请求下，哪一个句柄去接收和响应。句柄可以是一个函数、过程或者方法，只要能够接收特定类型的请求并做出反应即可。路由主要依据请求的方法“HTTP request method”和请求内容包括的路径信息来决定的。一个路由就是“HTTP method”方法、路径和句柄的组合。路由需要在服务器启动并开始监听请求或调度信号之前注册到服务器上，比如：

``` swift
var routes = Routes()
routes.add(method: .get, uri: "/path/one", handler: { request, response in
    response.setBody(string: "路由句柄已经收到")
    response.completed()
})
server.addRoutes(routes)
```

一旦Perfect服务器接收到一个HTTP请求，服务器会将请求对象传递给任何已注册的过滤器。这些过滤器根据需要可能会修改请求对象，因此有可能会影响路由的目标路径。服务器在请求过滤完成之后会搜索匹配当前请求方法和路径的路由。如果路由被请求成功匹配，则服务器会同时将请求和响应对象都发给路由句柄。如果特定请求没有找到合适的路由，则服务器会给客户端浏览器发一个“404 Not Found”错误。

### 创建路由

路由API函数是[PerfectHTTP](https://github.com/PerfectlySoft/Perfect-HTTP)项目的一个组成部分。要使用该路由系统，首先要在源代码开始部分导入库函数`import PerfectHTTP`。

在增加路由之前，首先要准备好适当的句柄函数。句柄函数必须能够同时接纳HTTPRequest请求对象和HTTPResponse响应对象，并且能够为响应对象创建适当的内容：

``` swift
/// 用于接受请求并根据请求创建响应内容的句柄函数格式。
public typealias RequestHandler = (HTTPRequest, HTTPResponse) -> ()
```
在路由期间请求都是一直处于活动状态，直到句柄返回说请求已经完成为止。确定请求完成是通过调用`HTTPResponse.next()` 或者 `HTTPResponse.completed()`函数而来。在Perfect中请求的处理完全是异步的，因此句柄函数可以返回、或者转入新线程、或者执行其它任何一种异步操作。

请求句柄可以使用链式组合操作，任何其中一个请求的URI资源都可以在其路径中识别多个句柄。各个句柄会按照顺序执行，每个句柄都有机会选择继续执行请求或者终止该请求。

该请求将一直保持活动状态，直到有`HTTPResponse.completed()`方法被调用。如果没有更多句柄需要执行，则调用`.next()`的结果与`.complete()`的结果是等价的。一旦一个响应被标记为结束，如果响应内还包含有待发出的消息头或消息体数据，则这些消息内容都会被服务器返还给客户端浏览器。

注册路由需要在服务器启动监听前追加到`Routes`路由表对象中。一旦路由表创建后，一个或多个路由都可以使用路由表的`add`方法追加进去。路由表提供以下函数

``` swift
public struct Routes {
    /// 不需要任何基本URL的构造函数。
    public init(handler: RequestHandler? = nil)
    // 使用基本URL初始化的构造函数
    public init(baseUri: String, handler: RequestHandler? = nil)
    /// 将另外一个路由表的内容全部加入到当前路由表中。
    public mutating func add(routes: Routes)
    /// 根据指定的HTTP方法、URI和一个路由句柄来创建一个新路由。
    public mutating func add(method: HTTPMethod, uri: String, handler: RequestHandler)
		/// 根据指定的HTTP方法、一组URI和一个路由句柄来创建一个新路由。
    public mutating func add(method: HTTPMethod, uris: [String], handler: RequestHandler)
    /// 根据指定的URI和一个路由句柄来创建路由，
    /// 不区分究竟是GET还是POST方法。
    public mutating func add(uri: String, handler: RequestHandler)
		/// 根据指定的一组URI和一个路由句柄来创建路由，
    /// 不区分究竟是GET还是POST方法。
    public mutating func add(uris: [String], handler: RequestHandler)
    /// 将一个路由对象增加到当前路由表中。
    public mutating func add(_ route: Route)
}
```

路由表可以从一个基本URI进行初始化。该基本的URI会在任何追加到路由表的新路由路径之前。比如，您可以用一个函数的第一个版本初始化一个路由表对象，用于初始化的基本URI为“/v1”。之后在这个路由表中新增的路由都会带有“/v1”前缀。同样，路由表对象也可以被加入到另一个路由表对象中去，因此每一个原路由表中的路由都会按照这种方法增加前缀。以下例子显示了同一个API函数对应两个不同版本的两个路由表的创建过程，两个版本的唯一区别在于其接口点不同：

``` swift
var routes = Routes()
// 为程序接口API版本v1创建路由表
var api = Routes()
api.add(method: .get, uri: "/call1", handler: { _, response in
    response.setBody(string: "程序接口API版本v1已经调用")
    response.completed()
})
api.add(method: .get, uri: "/call2", handler: { _, response in
    response.setBody(string: "程序接口API版本v2已经调用")
    response.completed()
})

// API版本v1
var api1Routes = Routes(baseUri: "/v1")
// API版本v2
var api2Routes = Routes(baseUri: "/v2")

// 为API版本v1增加主调函数
api1Routes.add(routes: api)
// 为API版本v2增加主调函数
api2Routes.add(routes: api)
// 更新API版本v2主调函数
api2Routes.add(method: .get, uri: "/call2", handler: { _, response in
    response.setBody(string: "程序接口API版本v2已经调用第二种方法")
    response.completed()
})

// 将两个版本的内容都注册到服务器主路由表上
routes.add(routes: api1Routes)
routes.add(routes: api2Routes)
```

每个路由对象都可以追加一个可选的句柄。如果该部分路径与另一个后续句柄的路径匹配，则会调用该句柄。

比如，一个“/v1”版本的的句柄可能要求客户必须登录并获得有效用户身份认证之后才能使用。凡是在该句柄后追加的新路由，都必须通过该句柄的用户身份验证才能继续执行：

``` swift
var routes = Routes(baseUri: "/v1") {
	request, response in 
	if authorized(request) {
		response.next()
	} else {
		response.completed(.unauthorized)
	}
}
routes.add(method: .get, uri: "/call1") { 
	_, response in
	response.setBody(string: "API CALL 1").next()
}
routes.add(method: .get, uri: "/call2") { 
	_, response in
	response.setBody(string: "API CALL 2").next()
}
```

上述参考范例中，任何访问“/v1/call1”或“/v1/call2”的操作都首先执行“/v1”路由的身份认证。

通过这种方法，路由可以不断地加深。直接设置在上述路由对象上的句柄是被系统视为“非末端路由”节点。也就是说，用户不能直接调用该“中间路由”；反过来相对于“中间路由”而言，只有全路径完全匹配后的“末端路由”才可以访问。比如，如果有句柄“/v1/users”和“/v1/users/foo”，则执行“/v1/users/foo”是可以的，但是“/v1/users”不能直接访问。

### 增加服务器路由

在Perfect项目中，无论是HTTP 1.1服务器还是FastCGI服务器都支持路由。详见[HTTPServer](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HTTPServer.md)查看路由编写的方法。

### 路由变量

URI路由还能够包括不同的变量组件。每个变量组件是通过一个块`{ }`声明的。在程序块中是变量名称。每个变量名称都可以使用出了括号`}`之外的任何字符。变量名有点像单功能通配符一样，这样就可以匹配任何符合变量模式的路径。匹配该模式的URL能够通过`HTTPRequest.urlVariables`字典查询变量值。该字典是`[String:String]`类型。URI变量是用于处理动态请求的好方法。比如，一个包含用户id的URL可以用该方法实现相关请求的用户管理。

比如，给定URI`/foo/{bar}/baz`，则如果有URL `/foo/123/baz`将匹配这个模式并将变量值替换到`HTTPRequest.urlVariables`字典中去，只要字典内包括关键词“bar”，则访问该字典会得到字符串值“123”。

### 通配符

通配符就是一个能够代表一个或多个字符的特殊符号。路由内能够使用通配符简化对URI路径的完整说明。

通配符能够匹配URI中的任何内容，因此可以将一组URI定向到同一个句柄上。通配符由一个或多个星号组成。一个星号可以出现在URI的任何位置，用于代表URI的局部内容。双星号通配符（也称为结尾通配符）只能用于URI结尾。双星号用于匹配URI从星号开始至结尾的所有内容。

形如`/foo/*/baz`的URI可以匹配以所有URL：

```
/foo/123/baz
/foo/bar/baz
```

形如`/foo/**` 的URI可以匹配以所有URL：

```
/foo/bar/baz
/foo
```

形如`/**`的URI将匹配所有的HTTP请求。

结尾通配符能够匹配从通配符开始的所有以之前内容的URI，并替换路径中的对应匹配内容到`HTTPRequest.urlVariables`字典中。通过访问全局变量`routeTrailingWildcardKey`即可获悉结尾通配符究竟获得了什么样的字符串值。比如，给定路径URI“/foo/** ”和一个真正的URI请求“/foo/bar/baz”，那么以下的程序判断就是真值：

``` swift
request.urlVariables[routeTrailingWildcardKey] == "/bar/baz"
```

### 优先级/路由顺序

因为URI的路由可能存在潜在矛盾，因此路由文本、通配符和路由变量是通过特定顺序来进行检查的：

1. 带变量路由路径
2. 静态文本路径
3. 通配符路径
4. 结尾通配符

### 隐式结尾通配符

当服务器文档根目录`.documentRoot`属性被设置后，服务器会自动将一个结尾通配符`/**`自动路由到指定目录的静态内容上去。比如，将文档根目录设置为“./webroot”会允许服务器从该目录中读取静态文件用于完成请求响应。

### 更多信息

关于URL路由的更多信息，请参见[URL路由](https://github.com/PerfectlySoft/PerfectExample-URLRouting)程序案例
