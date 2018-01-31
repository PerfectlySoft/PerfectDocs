# Sessions 会话

Perfect 提供了包括 PostgreSQL、MySQL、SQLite3 和 CouchDB 在内的各类数据库，并包括用于开发会话交互过程的临时内存管理机制，并很快会提供 MongoDB 和 Redis 为后台的会话管理。

会话管理是所有Web应用环境的基础程序，提供了验证、临时变量存储、交易数据等等，比如购物车数据模型。

基本的会话系统设计方案是每当用户通过浏览器“访问”网站时，服务器会给用户一个临时的票据（即token，传统翻译为“令牌”，本文采用“票据”这一中文名称）并暂时保存在服务器上，即一串特殊的序列号标识，用以区别每一个不同的客户，在服务器／浏览器之间互相传递。浏览器端一般使用cookie进行本地文件方式保存票据，或者通过JSON格式反复在服务器／浏览器之间互相传递。因此票据可以根据浏览器端的编程方式，采用cookie缓存或者采用“持有”的方式进行存储（比如嵌入脚本页面，在提交请求时附加票据——已经超出了本文的范围）

为了管理并区分来自同一IP地址的不同客户请求，会话有一个非常重要的概念，即票据的时效性。一旦用户失去交互的活跃性超过一定时间（即在一定时间内没有与服务器有信息交互），则票据就失效了。这种方式提高了数据的安全性。

Perfect 会话机制实现了存储每个会话的创建时间、最后一次访问时间，以及时效期限。如果“最后一次访问时间” 加上 “超时限制” 小于当前时间，则会话过期。

每个会话对象包含以下属性：

* **token** - 票据（令牌）识别码，即会话唯一识别码
* **userid** - 可选项，用户身份标识字符串
* **created** - 会话创建时的整型时间戳，单位是秒
* **updated** - 会话最后一次访问时的整型时间戳，单位是秒
* **idle** - 时效期限：整数，单位是秒。即允许客户在这个时间范围内处于不活动状态，在此期间不算过期。
* **data** - 数据：一个[String:Any]类型字典，可以JSON格式进行存储。用于存储简单的临时信息。
* **ipaddress** - 会话创建时客户端的 IPv4 或 IPv6 地址。可选项，用于校验会话安全性，防范会话欺诈。
* **useragent** - 会话创建时请求头数据部分的 User Agent 用户代理字符串。可选项，用于校验会话安全性，防范会话欺诈。

## 代码示范

以下模块包含示范代码，每个范例都有各自的使用说明：

* [基于内存变量的会话管理](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)
* [基于PostgreSQL数据库的会话管理](https://github.com/PerfectExamples/Perfect-Session-PostgreSQL-Demo)
* [基于MySQL数据库的会话管理](https://github.com/PerfectExamples/Perfect-Session-MySQL-Demo)
* [基于SQLite数据库的会话管理](https://github.com/PerfectExamples/Perfect-Session-SQLite-Demo)
* [基于CouchDB数据库的会话管理](https://github.com/PerfectExamples/Perfect-Session-CouchDB-Demo)

## 安装

如果希望使用基于内存变量的会话管理，请在您的项目中修改Package.swift 文件并增加以下内容：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session.git", majorVersion: 3)
```

### 基于数据库的会话管理所需驱动

Redis:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-Redis.git", majorVersion: 3)
```

PostgreSQL:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-PostgreSQL.git", majorVersion: 3)
```

MySQL:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MySQL.git", majorVersion: 3)
```

SQLite3:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-SQLite.git", majorVersion: 3)
```

CouchDB:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-CouchDB.git", majorVersion: 3)
```

MongoDB:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MongoDB.git", majorVersion: 3)
```

## 配置

`SessionConfig` 结构包含了可以自行定义的会话管理设置：

``` swift
// 会话名称
// 这个名称也会成为浏览器cookie的名称
SessionConfig.name = "PerfectSession"

// 允许处于零活动连接的时间限制，单位是秒
// 86400 秒表示一天。
SessionConfig.idle = 86400

// 可选项，设置 Cookie作用域
SessionConfig.cookieDomain = "localhost"

// 可选项，锁定创建时用的 IP 地址，即在会话过程中不允许 IP 地址发生变化，否则视为欺诈。默认为 false
SessionConfig.IPAddressLock = true

// 可选项，锁定创建时所用的用户代理，即服务器在会话过程中不允许用户代理发生变化，否则视为欺诈。默认为 false
SessionConfig.userAgentLock = true

// 指定采用 CouchDB 作为内存管理后台
// 指定 CouchDB 内用于管理会话的数据库名称
SessionConfig.couchDatabase = "sessions"

// 指定采用 MongoDB 作为内存管理后台
// 指定 MongoDB 内用于管理会话的集合名称
SessionConfig.mongoCollection = "sessions"

```

如果要改变上述会话配置结构的变量值，则必须在定义驱动之前完成配置。

### 锁定 IP 地址和 User Agent 用户代理

如果 `SessionConfig.IPAddressLock` 或 `SessionConfig.userAgentLock` 被设置为真，则服务器会在整个会话过程中不允许客户端更换 IP 地址和用户代理。

这种方式增强了会话安全性，能够在一定程度上防止会话标识字符串被盗用后形成会话欺诈。

⚠️请注意⚠️ 如果锁定了 IP 地址，那么用户在切换 WIFI 或者有线以太网时，则会话会被关闭，用户必须重新登录。

### 定义会话驱动

每个会话管理机制都有其独特的驱动实现，并且为存储进行了优化，因此必须在服务器启动`server.start()`调用之前设置会话驱动和相应的 HTTP 过滤器。

在您的服务器主进程 main.swift 中，只要是修改了 `SessionConfig`的配置，请务必采取以下类似操作：

``` swift 
// 初始化服务器
let server = HTTPServer()

// 定义会话驱动
let sessionDriver = SessionMemoryDriver()

// 增加对应的过滤器，用于在路由请求之前和之后执行相关操作。
server.setRequestFilters([sessionDriver.requestFilter])
server.setResponseFilters([sessionDriver.responseFilter])
```
在 Perfect 架构中，过滤器相当于某种“中间件”。“请求”过滤器用于截获HTTP请求，并抽取票据标识代码，根据票据访问后台数据库并提取会话信息，然后把会话信息转交给服务器应用。而“响应”过滤器则是在服务器即将向浏览器写会数据之前，保存当前会话信息，并把票据更新、交回给浏览器。

详见以下“数据库驱动配置选项详解”。

### 访问会话数据

过滤器会把会话的 `token`（票据）、`userid`（用户标识）和`data`（用户数据）属性追加到`request`对象中，并传递给所有处理器句柄。其中，`userid`和`data`这两个用户属性都是可以进行读写操作的，并会自动保存到响应过滤器中。

参考以下简单的处理句柄代码示范：

``` swift
// 定义一个“索引”句柄
open static func indexHandlerGet(request: HTTPRequest, _ response: HTTPResponse) {
	// 从 TurnstileCrypto 加密函数库中产生一个随机数
	let rand = URandom()
	
	// 增加一些用于展示的随机数
	request.session.data[rand.secureToken] = rand.secureToken

	// 展示如何将当前会话数据转储为一个 JSON 字符串
	let dump = try? request.session.data.jsonEncodedString()

	// 用于展示会话票据标识代码的简单HTML页面，以及当前会话数据的JSON字符串
	let body = "<p>您的会话票据标识为：<code>\(request.session.token)</code></p><p>会话数据：<code>\(dump)</code></p>"

	// 发回响应
	response.setBody(string: body)
	response.completed()
}

```

获取当前会话票据标识代码：

``` swift
request.session.token
```

读取用户身份标识字符串：

``` swift
// 读取：
request.session.userid

// 设置：
request.session.userid = "MyString"
```

访问会话数据：

``` swift
// 读取：
request.session.data

// 写入
request.session.data["keyString"] = "Value"
request.session.data["keyInteger"] = 1
request.session.data["keyBool"] = true

// 读取特定内容
if let val = request.session.data["keyString"] as? String {
	let keyString = val
}
```


## 数据库驱动选项

### Redis

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-Redis.git", majorVersion: 3)
```

设置数据库服务器连接参数：

``` swift
RedisSessionConnector.host = "localhost"
RedisSessionConnector.port = 5432
RedisSessionConnector.password = "secret"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionRedisDriver()
```

### PostgreSQL

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-PostgreSQL.git", majorVersion: 3)
```

设置数据库服务器连接参数：

``` swift
PostgresSessionConnector.host = "localhost"
PostgresSessionConnector.port = 5432
PostgresSessionConnector.username = "username"
PostgresSessionConnector.password = "secret"
PostgresSessionConnector.database = "mydatabase"
PostgresSessionConnector.table = "sessions"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionPostgresDriver()
```

### MySQL

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MySQL.git", majorVersion: 3)
```

设置数据库服务器连接参数：

``` swift
MySQLSessionConnector.host = "localhost"
MySQLSessionConnector.port = 3306
MySQLSessionConnector.username = "username"
MySQLSessionConnector.password = "secret"
MySQLSessionConnector.database = "mydatabase"
MySQLSessionConnector.table = "sessions"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionMySQLDriver()
```

### SQLite

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-SQLite.git", majorVersion: 3)
```

设置数据库服务器连接参数：

``` swift
SQLiteConnector.db = "./SessionDB"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionSQLiteDriver()
```

### CouchDB

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-CouchDB.git", majorVersion: 3)
```

配置用于保存会话的CouchDB数据库名称

``` swift
SessionConfig.couchDatabase = "perfectsessions"
```

设置数据库服务器连接参数：

``` swift
CouchDBConnection.host = "localhost"
CouchDBConnection.username = "username"
CouchDBConnection.password = "secret"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionCouchDBDriver()
```

### MongoDB

修改Package.swift并增加依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MongoDB.git", majorVersion: 3)
```

配置用于保存会话的Mongo数据库名称

``` swift
SessionConfig.mongoCollection = "perfectsessions"
```

设置数据库服务器连接参数：

``` swift
MongoDBConnection.host = "localhost"
MongoDBConnection.database = "perfect_testing"
```

创建会话驱动实例：

``` swift
let sessionDriver = SessionMongoDBDriver()
```