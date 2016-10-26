# Turnstile 认证层

Turnstile 是一个受[Apache Shiro](http://shiro.apache.org/)启发的、为Swift定制的安全框架。Turnstile 不但可以在您的应用程序中的管理用户账号，也可以同样在服务器后台使用相同的账号管理功能。

基本函数库基础是在程序员Edward Jiang的Stormpath上构造，虽然原有程序并非一个软件框架，但是在此基础上的工作为集成到Perfect程序提供了极大的便利。

根据目标数据源的不同，您也可以自行根据基本框架结构进行二次开发实现更多数据源的连接应用。但是基本模式就是为用户进行Web登录后采用JSON进行交互、登记注册和检查验证。

为实现上述应用，Turnstile “域对象”的内容被引入和编写，还包括会话驱动（Session Driver）。网页内会话（Web Session）过程的数据保存是通过Cookies完成，而JSON会话是通过消息头数据交互内容的token令牌控制完成。

Turnstile 应用的文档可以直接从[Stormpath/Turnstile](https://github.com/stormpath/Turnstile) 资源库获取。

## Turnstile-Perfect 数据源集成

Perfect 支持某些数据源集成，您可以在此基础之上根据项目需要效仿该模式进行二次开发。

* [Perfect-Turnstile-PostgreSQL](https://github.com/PerfectlySoft/Perfect-Turnstile-PostgreSQL) --  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo)
* [Perfect-Turnstile-SQLite](https://github.com/PerfectlySoft/Perfect-Turnstile-SQLite) --  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo)

### 安装

在您的 Package.swift 文件中，请增加以下依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-PostgreSQL.git", majorVersion: 0, minor: 0)
// or
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-SQLite.git", majorVersion: 0, minor: 0)
```

### JSON 路由

注意，如果要使用本函数库，则以下基本路由必须被设置如下：

```
POST /api/v1/login (包含了表单需要的用户名密码)
POST /api/v1/register (包含了表单需要的用户名密码)
GET /api/v1/logout
```

### 浏览器测试路径

为了在浏览器内测试，请使用以下的路由：

```
http://localhost:8181
http://localhost:8181/login
http://localhost:8181/register
```

上述路由存放在项目webroot目录下，并使用Mustache模板格式存储。

Mustache 文档样例请参考[https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo) 或者 [https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo)

### 创建一个带用户身份验证的HTTP服务器

有了Turnstile，设置一个带用户身份验证的HTTP服务器是非常容易的。参考下列服务器主程序代码，更详细的逐步说明请见下文：

``` swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

import StORM
import PostgresStORM
import PerfectTurnstilePostgreSQL

// 或者，如果您的数据库是SQLite，则请选择下面的库函数
// import SQLiteStORM
// import PerfectTurnstileSQLite

// 如果您需要在终端上调试SQL，请将下面一行的注释去掉
//StORMdebug = true

// 下面这个变量在后续的Real对象程序脚本中将用于用户身份验证
let pturnstile = TurnstilePerfectRealm()

// 设置连接参数
// 替换为您自己的信息
connect = PostgresConnect(
    host: "localhost",
    username: "perfect",
    password: "perfect",
    database: "perfect_testing",
    port: 32769
)
// 如果要连接SQLite，请使用：
// connect = SQLiteConnect("./authdb")

// 设置认证数据表
let auth = AuthAccount(connect!)
auth.setup()

// 连接到令牌数据区
tokenStore = AccessTokenStore(connect!)
tokenStore?.setup()

// 创建 HTTP 服务器。
let server = HTTPServer()

// 注册路由和具柄
let authWebRoutes = makeWebAuthRoutes()
let authJSONRoutes = makeJSONAuthRoutes("/api/v1")

// 在服务器上注册路由
server.addRoutes(authWebRoutes)
server.addRoutes(authJSONRoutes)

// 其他的路由内容也可以在这里准备好
var routes = Routes()
// routes.add(method: .get, uri: "/api/v1/test", handler: AuthHandlersJSON.testHandler)

// 追加其他路由
server.addRoutes(routes)

// 增加认证路由
var authenticationConfig = AuthenticationConfig()
authenticationConfig.include("/api/v1/check")
authenticationConfig.exclude("/api/v1/login")
authenticationConfig.exclude("/api/v1/register")

let authFilter = AuthFilter(authenticationConfig)

// 注意如果不同过滤器处于同等优先级过滤器，注意程序的执行顺序将影响过滤器处理的前后顺序
server.setRequestFilters([pturnstile.requestFilter])
server.setResponseFilters([pturnstile.responseFilter])

server.setRequestFilters([(authFilter, .high)])

// 设置监听端口 8181
server.serverPort = 8181

// 设置服务器静态文件根目录
server.documentRoot = "./webroot"

do {
	// 启动 HTTP 服务器
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("网络异常： \(err) \(msg)")
}

```

#### 基本要求

首先定义一下“域对象 Realm” —— 这是 Turnstile 定义的用户身份验证处理的管理方法。其实现方法是通过连接一个数据源（比如SQLite）—— 其实各个数据源的连接方法都大同小异， 您可以自行参考，在此基础之上进行扩展。

``` swift
let pturnstile = TurnstilePerfectRealm()
```

定义连接参数。如果您使用 PostgreSQL：

``` swift
// 请替换为您自己的配置
connect = PostgresConnect(
    host: "localhost",
    username: "perfect",
    password: "perfect",
    database: "perfect_testing",
    port: 32769
)
```

或者，配置好SQLite的数据库文件位置：

``` swift
connect = SQLiteConnect("./authdb")
```

定义并初始化身份认证表：

``` swift
let auth = AuthAccount(connect!)
auth.setup()
```

连接到令牌数据区：

``` swift
tokenStore = AccessTokenStore(connect!)
tokenStore?.setup()
```

创建 HTTP 服务

``` swift
let server = HTTPServer()
```

登记并注册路由句柄：

``` swift
let authWebRoutes = makeWebAuthRoutes()
let authJSONRoutes = makeJSONAuthRoutes("/api/v1")

server.addRoutes(authWebRoutes)
server.addRoutes(authJSONRoutes)
```

为身份验证表配置访问路由：

``` swift
var authenticationConfig = AuthenticationConfig()
authenticationConfig.include("/api/v1/check")
authenticationConfig.exclude("/api/v1/login")
authenticationConfig.exclude("/api/v1/register")

let authFilter = AuthFilter(authenticationConfig)
```

输入路由的时候可以一个一个分别录入，或者整个儿做成一个数组一起放进去，可以把需要包含的路径和不希望包含的路径都分别进行登记处理。下一版的函数库允许使用通配符处理路径。

增加请求/响应的过滤器。注意如果不同过滤器的优先等级相同，则一定要处理好写程序的顺序，因为这种情况下程序顺序决定了最终路由表内过滤器的前后处理顺序：

``` swift
server.setRequestFilters([pturnstile.requestFilter])
server.setResponseFilters([pturnstile.responseFilter])

server.setRequestFilters([(authFilter, .high)])
```

随后可以设置监听端口、静态文件并最终启动服务器：

``` swift

// 设置监听端口 8181
server.serverPort = 8181

// 设置服务器静态文件根目录
server.documentRoot = "./webroot"

do {
	// 启动 HTTP 服务器
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("网络异常： \(err) \(msg)")
}
```
