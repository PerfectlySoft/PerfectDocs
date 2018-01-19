# HTTP 服务器

**⚠️注意⚠️** 本文内容为试验性版本，在最终稳定版发布之前，可能随时会内容进行调整。

本文描述了启动 Perfect HTTP 服务器的三种方法。每种方法适用于不同的场合各有不同。

第一种方法是用Swift 字典或者 JSON文件构造配置内容并启动服务器。第二种方法是直接在服务器用Swift源代码编写配置信息。这样的好处是能够在编译阶段进行语法检查，避免运行时出现低级错误。第三种方法是启动HTTPServer对象后挂起，完成配置和填写必要属性后再手工启动。

HTTP服务器能够通过`HTTPServer`命名空间进行配置。每一个 Perfect HTTP Server 包括至少一个名称、监听端口、一个或多个请求处理器句柄，以及不同的请求／响应过滤器。此外，HTTPS传输加密服务器的配置还需要更多信息，比如证书和密钥路径。

启动服务器后，您可以选择等待服务器直到退出（很难等到这种情况，除非服务器崩溃），或者选择针对每个服务器获取一个`LauchContext`的启动上下文环境对象，这样就可以单独针对某个服务器，进行等候或终止的操作。

## HTTP 服务器配置文件

使用结构化配置文件可以使得一个或者多个Perfect HTTP服务器进行配置并启动。配置信息包括基本设置，比如监听端口和绑定地址，也包括路由处理器句柄等等更高级的配置处理。如果您正在使用 Linux 系统，那么在编译时请务必注意手工填写下列编译器标志：

```
swift build -Xlinker --export-dynamic
```
如果您希望应用本文所描述的配置系统，则只有Linux需要该编译选项。


您可以调用静态方法`HTTPServer.launch`来控制服务器的启动，其参数可以选择为一个 JSON 格式的配置文件路径、或者配置文件的`File`文件对象；还可以将一个 Swift 的字典变量（`[String:Any]`类型）作为配置参数传递给这个启动方法。

配置内容将会被应用到启动指定的服务器上。

``` swift
public extension HTTPServer {
	// 采用配置文件的路径启动服务器
	public static func launch(wait: Bool = true, configurationPath path: String) throws -> [LaunchContext]
	// 采用配置文件的文件对象启动服务器
	public static func launch(wait: Bool = true, configurationFile file: File) throws -> [LaunchContext]
	// 采用配置字典启动服务器
	public static func launch(wait: Bool = true, configurationData data: [String:Any]) throws -> [LaunchContext]
}
```

默认情况下`wait` 参数意味着主进程要等待所有服务器终止退出。如果等待参数为`false`，则执行后立刻返回一个 `LauchContext` 数组，用户可以监控其状态，或者终止指定的服务器。当然多数情况下直接启动就可以，不用包括这个等待参数：

``` swift
do {
	try HTTPServer.launch(configurationPath: "/path/to/perfecthttp.json")
} catch {
	// 处理服务器异常
}
```
⚠️请注意⚠️ 配置文件您可以自由命名，但是一定要以`.json`为文件扩展名。因为 Perfect 以后可能会识别其他的配置文件格式，因此扩展名会非常重要，决定了文件内容的识别方法。

从 JSON 数据解码之后，配置文件的顶层数据结构会包含一个“serves”（服务器）数组，实际上是一个字典&lt;String:Any&gt;条目名称就是服务器名称，而变量值就是对应该服务器的配置信息。

``` swift
[
	"servers":[
		[…],
		[…],
		[…]
	]
]
```

参考配置文件示例如下。注意其配置的条目和取值会在本文随后内容进行解释。

``` swift
[
	"servers":[
		[
			"name":"localhost",
			"port":8080,
			"routes":[
				[
					"method":"get",
					"uri":"/**",
					"handler":"PerfectHTTPServer.HTTPHandler.staticFiles",
					"documentRoot":"/path/to/webroot"
				],
				[
					"methods":["get", "post"],
					"uri":"/api/**",
					"handler":"PerfectHTTPServer.HTTPHandler.redirect",
					"base":"http://other.server.ca"
				]
			]
		]
	]
]
```

可选项目如下：

### name（命名）:

**必填参数** 服务器名称。通常您可以使用服务器的域名作为这个名称。应用程序可以利用该名称确定指向服务器的 URL 。另外，服务器允许重名，比如您的同一主机上有三个不同的端口，则这三个服务器可以采用同样的名称。

该属性对应于`HTTPServer`对象类的属性`HTTPServer.serverName`。

### port（端口）:

**必填参数** 即服务器监听端口。其端口数值范围是 0 到 65535 之间的正整数。如果您希望使用0到1024之间的标准TCP端口，则需要系统管理员权限启动服务器。详见 *runAs* 选项，用于绑定端口前后切换用户身份。

对应的`HTTPServer`类对象属性： `HTTPServer.serverPort`。

### address（地址）:

**可选参数** 即服务器绑定的目标IP地址。 如果不设置该地址，则服务器自动绑定到`0.0.0.0`，意味着能够接受该服务器上所有网卡（IP地址）的请求。

对应的`HTTPServer`类对象属性： `HTTPServer.serverAddress`.

### runAs（切换用户）:

**可选参数** 即以其他用户身份运行。典型用法是，启动时为了监听1024以内的端口（比如标准的80端口），则必须以root管理员身份执行绑定，而执行绑定完成后，可以切换到普通用户。这种做法的目的是首先能够使用这种需要授权的端口，而在授权之后又不会在受到网络攻击时导致过大的权限被滥用。

对应的`HTTPServer`类对象属性： `HTTPServer.runAsUser`.

### routes（路由）:

**可选参数** 即路由管理。该参数是一个字典`[String:Any]`。每个路由都指向一个特定的HTTP请求处理器。详见[路由管理](routing.md)。

每个路由都包含一个或多个HTTP方法，也就是一个URI资源和一个能够返回请求句柄`RequestHandler`的函数名。

条目名称包括“method”（单个HTTP方法，比如GET）或“methods”（多个HTTP方法）、“uri”资源路径和“handler”处理器。除了“methods”是一个字符串数组之外，其他每个条目对应的取值都是一个字符串。如果不设置method条目，则默认所有方法都适用与该URI的配套处理器。

根据函数调用的需要还可以追加更多的条目及其配套取值，用于配置路由处理器的行为。比如，`staticFiles` 静态文件处理器需要一个“documentRoot”文档根目录的变量，用于配制服务器本地静态页面文件存储的目录。

Perfect 包含了大多数常见HTTP任务的功能，比如处理静态页面文件或者客户端请求重定向。下面的例子展示了一个监听8080端口的服务器和两个路由处理器，一个是静态文件，另外一个是路由重定向。

``` swift
[
	"servers":[
		[
			"name":"localhost",
			"port":8080,
			"routes":[
				[
					"method":"get",
					"uri":"/**",
					"handler":"PerfectHTTPServer.HTTPHandler.staticFiles",
					"documentRoot":"/path/to/webroot"
				],
				[
					"methods":["get", "post"],
					"uri":"/api/**",
					"handler":"PerfectHTTPServer.HTTPHandler.redirect",
					"base":"http://other.server.ca"
				]
			]
		]
	]
]
```
对应的`HTTPServer`类对象属性： `HTTPServer.addRoutes`.

#### 追加定制请求处理器

尽管 Perfect 的内建页面处理器已经非常完善了，相信大多数程序员还是喜欢为他们的服务器定制功能。配置条目`"handler"`用于指向您自行定义的处理器函数（路由句柄），只要该函数符合标准，能够返回在路由匹配时返回对应的 `RequestHandler`请求句柄。

有一点要注意，您的自定义处理器函数必须是静态类型 **static** ，并且返回值*必须是* `RequestHandler`类型。这些函数能够接收当前字典的配置，如同之前描述的静态文件`staticFiles`指向`"documentRoot"`一样。

另外函数名称也要符合标准。首先，是您的 Swift 模块名称，其次是内联结构或枚举的名称，最后是函数名称，并且将这三个名称用小数点符号连接起来，形成完整的一个命名空间名称。比如之前介绍的静态文件处理器，其名称就是`PerfectHTTPServer.HTTPHandler.staticFiles`，来源是服务器模块"PerfectHTTPServer"、在结构"HTTPHandler"中实现的方法"staticFiles"。

注意，如果您是从Swift源代码进行配置的，则不需要引用您的句柄名称，而是直接将函数本身作为句柄进行赋值（“过滤器”的管理配置也是这样）。

下面的例子说明了如何自定义一个路由处理器：

``` swift
public extension HTTPHandler {
	public static func staticFiles(data: [String:Any]) throws -> RequestHandler {
		let documentRoot = data["documentRoot"] as? String ?? "./webroot"
		let allowResponseFilters = data["allowResponseFilters"] as? Bool ?? false
		return {
			req, resp in
			StaticFileHandler(documentRoot: documentRoot, allowResponseFilters: allowResponseFilters)
				.handleRequest(request: req, response: resp)
		}
	}
}
```

注意：`HTTPHandler`结构是在`PerfectHTTPServer`中定义的一个抽象的命名空间，只能容纳类似的静态句柄（处理器）。

在编写您的处理器代码时还要注意，如果处理器需要特定的配置数据但是最终用户可能输入错误的数据或配置不完整时，为了面对这种情况，请务必在代码中增加异常处理，`throw`方法抛出错误。另外，还请注意抛出异常时最好要有准确而详细的错误信息字符串，以便最终用户排查错误和修改配置。如果处理器无法返回一个有效的`RequestHandler`请求句柄，则就应该立刻抛出一个异常。

### filters（过滤器）:

即过滤器。请求过滤器能够对请求数据进行预处理。比如，验证过滤器可以用于检查用户是否具有特定权限，如果没有就直接将页面重定向到提示用户访问受限。而响应处理器则是为返回的响应进行预处理，可以在最终的页面产生之前修改响应头和数据体。详见[请求/响应过滤器](filters.md)。

配置条目"filters"之下是一个数组结构，每个元素对应一个过滤器，而每个过滤器都有若干属性用于描述过滤器的功能作用和配置信息。其中，必须要填写的内容包括每个过滤器的类型和名称——"type"和"name"。其中类型的取值可以选择`"request"`（请求）或者`"response"`（响应），说明过滤器类型究竟是请求还是响应。除此之外，还包含一个`"priority"`字段，用于说明该过滤器的优先级，其取值为`"high"`（高）、`"medium"`（中）或`"low"`（低）。如果没有设置优先级，则默认为高优先级。

以下配置展示了两种过滤器，一个请求过滤器和一个响应过滤器。

``` swift
[
	"servers": [
		[
		"name":"localhost",
		"port":8080,
		"routes":[
			["method":"get", "uri":"/**", "handler":"PerfectHTTPServer.HTTPHandler.staticFiles",
			"documentRoot":"./webroot"]
		],
		"filters":[
			[
				"type":"request",
				"priority":"high",
				"name":"PerfectHTTPServer.HTTPFilter.customReqFilter"
			],
			[
				"type":"response",
				"priority":"high",
				"name":"PerfectHTTPServer.HTTPFilter.custom404",
				"path":"./webroot/404.html"
			]
		]
	]
]

```

过滤器工作方式和路由处理器相差无几，但是函数性质是不一样的。过滤器函数从字典`[String:Any]`变量中读取配置数据然后根据具体的过滤器类型返回`HTTPRequestFilter`（请求过滤器）或者`HTTPResponseFilter`（响应过滤器）。

``` swift
// 生成一个请求过滤器
public func customReqFilter(data: [String:Any]) throws -> HTTPRequestFilter {
	struct ReqFilter: HTTPRequestFilter {
		func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
			callback(.continue(request, response))
		}
	}
	return ReqFilter()
}

//创建一个响应过滤器
public func custom404(data: [String:Any]) throws -> HTTPResponseFilter {
	guard let path = data["path"] as? String else {
		fatalError("HTTPFilter.custom404(data: [String:Any]) 无法找到配置文件数据\"path\"。")
	}
	struct Filter404: HTTPResponseFilter {
		let path: String
		func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
			if case .notFound = response.status {
				do {
					response.setBody(string: try File(path).readString())
				} catch {
					response.setBody(string: "有错误发生但无法找到对应的报错页面 \(response.status)")
				}
				response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
			}
			return callback(.continue)
		}
		func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
			callback(.continue)
		}
	}
	return Filter404(path: path)
}
```

对应的`HTTPServer`类对象属性： `HTTPServer.setRequestFilters`, `HTTPServer.setResponseFilters`.

### 在请求/响应/过滤器之间共享信息

有时候很需要在HTTP的请求响应和过滤器之间共享必要的信息。比如，如果存在一个用于用户身份验证的过滤器，时刻监控所有请求，并从请求中的cookie或者JWT中提取登录凭证，并根据凭证取出用户代码即用户档案文件，然后应用到所有与该请求相关的响应中，从而构成一个典型的安全系统设计。

虽然HTTP是无状态的，并且过滤器和请求之间也不是透明的，但是Perfect HTTP服务器依然提供了一种间接方法用于在每个请求内部共享数据，使得整个系统设计更为紧凑高效：

``` swift
/// 认证中间层，用于监听所有请求，并假设请求内都必须包括有效的登录凭证
func filter(request: HTTPRequest, response: HTTPResponse,
                     callback: (HTTPRequestFilterResult) -> ()) {
  guard let token = request.param("token"), 
  let userId = token.decode(),
  let profile = database.load(userId) as? Profile else {
  	// 该请求不包含有效凭证
  	// 拒绝访问
  	response.status = .forbidden
  	response.completed
  	callback(.halt(request, response))
  }
  
  	// 用户档案已经提取，现在暂存到内存
  response.request.scratchPad["currentUserProfile"] = profile
  	
  // 触发后续请求
  callback(.continue(request, response))
}

routes.add(Route(method: .get, uri: "/api/aboutUser", handler: {
      request, response in
      guard let profile 
      	= response.request.scratchPad["currentUserProfile"] as? Profile
		else {
      		//something wrong
      	}
      	/// 现在可以直接使用用户档案变量profile
    }))
```

上述例子中，具有优先级的过滤器首先拦截了请求，并从请求中根据凭证提取用户数据，然后把用户资料传递给后续请求，因此实现了一个典型的集中登录管理中间件。


### tlsConfig（传输安全配置）:

即传输层安全协议配置信息。如果在配置中增加了`"tlsConfig"`条目，则服务器将尝试自动配置为一个HTTPS的认证服务器。TLS 配置字典应当包括如下关键信息：

* certPath - **必填项** 认证文件的完整路径
* keyPath - 可选字符串，用于保存密钥文件路径
* cipherList - 可选字符串数组，用于为服务器提供密码序列。
* caCertPath - 可选字符串，用于说明CA证书文件的详细路径
* verifyMode - 校验模式，可选字符串用于说明安全连接是如何被认证的。允许值包括（每个配置选项的具体解释超过本文范围，请自行查阅有关文档）：
	* none
	* peer
	* failIfNoPeerCert
	* clientOnce
	* peerWithFailIfNoPeerCert
	* peerClientOnce
	* peerWithFailIfNoPeerCertClientOnce

密码序列清单cipher list的默认值可以通过`TLSConfiguration.defaultCipherList`属性获得。

## HTTPServer.launch （服务器启动）

服务器启动`HTTPServer.launch`方法有数个变种，允许一个或多个服务器进行启动操作。这些函数能够把服务器对象的内部工作进行抽象处理，便于编程和操控。

最简单的启动方式是单个服务器启动：

``` swift
public extension HTTPServer {
	public static func launch(wait: Bool = true, name: String, port: Int, routes: Routes, runAs: String? = nil,
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
	public static func launch(wait: Bool = true, name: String, port: Int, routes: [Route], runAs: String? = nil,
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
	public static func launch(wait: Bool = true, name: String, port: Int, documentRoot root: String, runAs: String? = nil,
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
}
```

其他的启动函数可以启动一个或多个服务器，并返回每个服务器的`LaunchContext`对象。

``` swift
public extension HTTPServer {
	public static func launch(wait: Bool = true, _ servers: [Server]) throws -> [LaunchContext]
	public static func launch(wait: Bool = true, _ server: Server, _ servers: Server...) throws -> [LaunchContext]
}
```

最终被启动的`Server`对象，实际上是如下结构：

``` swift
public extension HTTPServer {
	public struct Server {
		public init(name: String, address: String, port: Int, routes: Routes, runAs: String? = nil,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(tlsConfig: TLSConfiguration, name: String, address: String, port: Int, routes: Routes, runAs: String? = nil,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(name: String, port: Int, routes: Routes, runAs: String? = nil,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(tlsConfig: TLSConfiguration, name: String, port: Int, routes: Routes, runAs: String? = nil,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])

		public static func server(name: String, port: Int, routes: Routes, runAs: String? = nil,
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func server(name: String, port: Int, routes: [Route], runAs: String? = nil,
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func server(name: String, port: Int, documentRoot root: String, runAs: String? = nil,
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func secureServer(_ tlsConfig: TLSConfiguration, name: String, port: Int, routes: [Route], runAs: String? = nil,
		                                requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                                responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
	}
}
```

以下代码展示了一些常规用法：

``` swift
// 启动一个服务器用于处理静态文件
try HTTPServer.launch(name: "localhost", port: 8080, documentRoot: "/path/to/webroot")

// 启动两个服务器，一个用于处理静态文件，另外一个用于处理API界面编程请求
let apiRoutes = [Route(method: .get, uri: "/foo/bar", handler: {
					req, resp in
					//在这里处理具体内容
				})
]
try HTTPServer.launch(
	.server(name: "localhost", port: 8080, documentRoot:  "/path/to/webroot"),
	.server(name: "localhost", port: 8181, routes: apiRoutes))

// 启动一个服务器，同时处理静态文件和可编程界面API
try HTTPServer.launch(name: "localhost", port: 8080, routes: [
	Route(method: .get, uri: "/foo/bar", handler: {
		req, resp in
		//在这里处理请求和响应
	}),
	Route(method: .get, uri: "/foo/bar", handler:
		HTTPHandler.staticFiles(documentRoot: "/path/to/webroot"))
	])

let apiRoutes = [Route(method: .get, uri: "/foo/bar", handler: {
	req, resp in
	//do stuff
})
]
// 启动一个安全服务器
try HTTPServer.launch(.secureServer(TLSConfiguration(certPath: "/path/to/cert"), name: "localhost", port: 8080, routes: apiRoutes))
```

`TLSConfiguration`用于配置服务器的安全传输层，定义如下：

``` swift
public struct TLSConfiguration {
	public init(certPath: String, keyPath: String? = nil,
	            caCertPath: String? = nil, certVerifyMode: OpenSSLVerifyMode? = nil,
	            cipherList: [String] = TLSConfiguration.defaultCipherList)
}
```

## LaunchContext （启动上下文环境）

如果`HTTPServer.launch`函数在调用中使用了 `wait: false`，则每个服务器的状态都可以在调用后进行检查，并允许终止服务器运行。

``` swift
public extension HTTPServer {
	public struct LaunchFailure: Error {
		let message: String
		let configuration: Server
	}

	public class LaunchContext {
		public var terminated: Bool
		public let server: Server
		public func terminate() -> LaunchContext
		public func wait(seconds: Double = Threading.noTimeout) throws -> Bool
	}
}
```

如果等待过程中，服务器启动失败并抛出错误，则错误信息将会被解析为对应的文字和配置并继续上抛。

## HTTPServer 服务器对象

`HTTPServer`对象可以被初始化、配置和手工启动。

``` swift
public class HTTPServer {
	/// 服务器用于存储静态页面文件的目录路径
	/// 如果该路径名称非空，则默认路由将指向该文档目录
	/// 用户只需要将页面文件手工拷贝到该目录下就可以对浏览器提供对应服务
	public var documentRoot: String
	/// 服务器监听端口
	public var serverPort: UInt16 = 0
	/// 服务器绑定的地址，默认为0.0.0.0，即允许从所有网卡（ip地址）访问服务器
	public var serverAddress = "0.0.0.0"
	/// 绑定端口之后允许切换到的普通用户名称
	public var runAsUser: String?

	/// 服务器规范名称
	/// 如果使用`HTTPRequest.serverName`属性，则必须要设置该项目
	public var serverName = ""
	public var ssl: (sslCert: String, sslKey: String)?
	public var caCert: String?
	public var certVerifyMode: OpenSSLVerifyMode?
	public var cipherList: [String]
	/// 初始化服务器
	public init()
	/// 为服务器增加路由
	public func addRoutes(_ routes: Routes)
	/// 设置请求过滤器。每个过滤器都要有权限配套
	/// 过滤器可以进行排序，高优先级的过滤器优先响应
	/// 如果两个同类型过滤器具有相同的优先级，则书写顺序决定了最终的调用优先顺序。
	public func setRequestFilters(_ request: [(HTTPRequestFilter, HTTPFilterPriority)]) -> HTTPServer
	/// 设置响应过滤器。每个过滤器都要有权限配套
	/// 过滤器可以进行排序，高优先级的过滤器优先响应
	/// 如果两个同类型过滤器具有相同的优先级，则书写顺序决定了最终的调用优先顺序。
	public func setResponseFilters(_ response: [(HTTPResponseFilter, HTTPFilterPriority)]) -> HTTPServer
	/// 启动服务器。除非出错或终止，否则将主进程阻塞于此
	public func start() throws
	/// 终止服务器运行并关闭监听端口。执行后主进程将越过阻塞继续后续程序执行。
	public func stop()
}
```

