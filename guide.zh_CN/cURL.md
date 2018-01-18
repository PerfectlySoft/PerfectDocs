# CURL联网传输

Perfect-CURL 函数库提供了Swift 语言版本的[curl](https://curl.haxx.se)支持。该组件采用Swift Package Manager软件包管理器进行编译。

## 编译

请确保安装并激活最新的Swift 4以上版本工具链，并在Package.swift文件中追加依存关系：
``` swift
.package(
	url: "https://github.com/PerfectlySoft/Perfect-Curl.git",
	from: "3.0.0"
	)
...
dependencies: ["PerfectCURL"]
```

## 使用方法

`import PerfectCURL`

本函数库使用了一个简单的访问/响应模型用于访问URL内容。首先请创建一个`CURLRequest`对象并根据需要进行配置，然后执行请求即可返回对应的响应对象，返回的响应对象类型为`CURLResponse`。

请求即可以是同步执行——阻塞当前线程并直到请求结束；或者是异步执行——从闭包或者Promise承诺对象中执行请求，这种方法意味着您需要做一些多线程操作，诸如状态查询或者结果等待之类。

### 创建请求

`CURLRequest`对象创建时可以带URL以及多个选项，或者可以首先创建一个空对象然后逐步配置。初始化方法如下：

``` swift
open class CURLRequest {
	// 用URL地址和选项集合初始化
	public convenience init(_ url: String, options: [Option] = [])
	/// 用URL地址及一系列不同的选项作为参数初始化
	public convenience init(_ url: String, _ option1: Option, _ options: Option...)
	/// 纯粹通过选项数组进行初始化
	public init(options: [Option] = [])
}
```

如以上代码所示，选项可以用数组作为参数，也可以用可变参数传递给初始化函数。只要是在请求执行之前，都可以通过直接增加或者设置`CURLRequest.options`属性修改这些配置选项。

### 配置请求

所有`CURLRequest` 选项都是通过 `CURLRequest.Option`枚举实现的。每个枚举实例都可以关联零个或多个选项值。比如，URL地址可以用选项`.url("https://httpbin.org/post")`转化成选项。目前大部分curl的选项都是用这种方法实现的，详见本文末尾的选项清单。

#### 表单数据

包括文件上传在内的表单数据，也都是采用类似上述选项的风格实现可以使用`.postField(POSTField)`、`.postData([UInt8])`和`.postString(String)` 来传递表单数据。`.postField`枚举类型可以在同一个访问实例中多次访问，对其中的不同字段和对应值可以进行多次反复修改；而`.postData` 和 `.postString` 在使用时则要注意因为这两种方式将覆盖访问请求的整个数据体，换言之一旦使用则之前写入的内容都会失效。另外请注意只要增加POST数据就会自动将请求改为POST方式。

`CURLRequest.POSTField` 数据结构定义如下：

``` swift
open class CURLRequest {
	public struct POSTField {
		/// 初始化表单字段变量，并设置变量名称、变量值和媒体类型。
		public init(name: String, value: String, mimeType: String? = nil)
		/// 初始化表单字段变量，并设置变量类型为待上传文件、将变量值设置为待上传文件路径；媒体类型为可选。
		public init(name: String, filePath: String, mimeType: String? = nil)
	}
}
```

以下代码展示了如何构造一个POST请求并追加数个字符串变量，以及一个待上传文件；此时表单媒体类型被自动设置为“multipart/form-data”。

``` swift
let json = try CURLRequest(url, .failOnError,
			               .postField(.init(name: "key1", value: "value1")),
			               .postField(.init(name: "key2", value: "value2")),
			               .postField(.init(name: "file1", filePath: testFile.path, mimeType: "text/plain")))
										.perform().bodyJSON
```

### 读取响应数据

调用`CURLRequest.perform`方法执行实际的网络请求。如果执行成功则会返回一个`CURLResponse`对象，用于处理返回的响应数据。如果失败则会自动抛出一个`CURLResponse.Error`错误。造成访问失败的原因有很多种，比如无法连接、连接超时、返回数据格式不正确，或者任何服务器返回了大于等于400代码的响应（如果设置了`.failOnError`选项）。如果没有设置`.failOnError`选项则只要服务器返回则无论状态代码是什么，则都会视为`CURLResponse`对象响应成功。

获得响应对象的方法主要有下列三个函数：

```swift
public extension CURLRequest {
	/// 同步执行请求；返回响应结果或者抛出错误
	func perform() throws -> CURLResponse
	/// 异步执行请求；请求完成后会调用回调函数，
	/// 并将响应结果传递给回调函数，或者抛出错误
	func perform(_ completion: @escaping (CURLResponse.Confirmation) -> ())
	/// 异步执行请求并返回一个Promise多线程承诺对象用于监控操作。
	func promise() -> Promise<CURLResponse>
}
```

在上述方法中，第一个`CURLRequest.perform`函数会在当前线程内同步执行请求。此时函数会阻塞当前线程直到访问成功返回或者抛出`CURLResponse.Error`错误。

第二个`CURLRequest.perform`方法能够异步执行请求，即利用后台线程执行。传递给这个函数的参数是一个回调函数（闭包），在执行完成时会用`CURLResponse.Confirmation`为参数通知回调函数确认响应结果；而在您的回调函数内调用这个确认参数的时候将得到`CURLResponse`响应或者自动抛出`CURLResponse.Error`错误。

第三种`CURLRequest.perform`调用方法会返回一个线程承诺对象`Promise<CURLResponse>`，用于继续链式操作，比如轮询或者等待请求结果。和其他的响应结果一致，该方法出错时会同样抛出`CURLResponse.Error`对象。Promise线程承诺对象可以从文档中[Perfect-Thread](http://www.perfect.org/docs/thread.html) 获得。

以下三个例子展示了上述三种方法的用法。每个例子都是执行请求并将响应结果从JSON字符串转化为一个[String:Any]字典：

• 同步请求并解析JSON数据：

``` swift
let url = "https://httpbin.org/get?p1=v1&p2=v2"
let json: [String:Any] = try CURLRequest(url).perform().bodyJSON
```

• 异步请求并解析JSON数据：

```swift
let url = "https://httpbin.org/post"
CURLRequest(url).perform {
	confirmation in
	do {
		let response = try confirmation()
		let json: [String:Any] = response.bodyJSON
		
	} catch let error as CURLResponse.Error {
		print("出错，响应代码为： \(error.response.responseCode)")
	} catch {
		print("致命错误： \(error)")
	}
}
```

• 异步请求并用Promise串联操作并解析JSON数据：

```swift
let url = "https://httpbin.org/get?p1=v1&p2=v2"
if let json = try CURLRequest(url).promise().then { return try $0().bodyJSON }.wait() {
	...
}
```

从效率上考虑上述三种调用方法的推荐顺序为：

1. 异步 `perform`
2. 异步 `promise`
3. 同步 `perform`

在流量负荷较大的服务器上请尽量使用异步操作。

#### 重置请求

`CURLRequest`对象是可以反复使用的，只要使用`.reset`函数就可以将该对象实例重新清理并回到初始化状态，即删除所有选项数据以及URL。为了方便使用，`.reset`方法还包括了一个选项参数用于重新配置。

该函数声明如下：

``` swift
public extension CURLRequest {
	/// 重置请求并清除所有选项数据，并可以直接输入新的选项数据
	func reset(_ options: [Option] = [])
	/// 重置请求并清除所有选项数据，并可以采用可变参数的方法直接输入新的选项数据
	func reset(_ option: Option, _ options: Option...)
}
```

### 响应数据

`CURLResponse`是专门用来访问响应数据的Swift对象类，访问方法可以是二进制字节数组、字符串或者从JSON解码得到的字典类型`[String: Any]`。此外，HTTP头的元数据信息也可以通过该对象获得具体内容。

访问方法是读取`CURLResponse`下列对象属性实现：

```swift
public extension CURLResponse {
	/// 直接读取返回结果的二进制字节数组
	public var bodyBytes: [UInt8]
	/// 将整个返回结果视为一个UTF-8编码字符串
	public var bodyString: String
	/// 以JSON解码字典的方式读取数据
	/// 如果非JSON数据则此项为空
	public var bodyJSON: [String:Any]
	/// 将返回的JSON数据进行结构性解码，返回可解码结构体
	/// 如果数据格式不符合要求则会出错
	public func bodyJSON<T: Decodable>(_ type: T.Type) throws -> T 
}
```

响应数据的其余部分可以通过调用`CURLResponse.get`命令并根据传递给该函数的枚举参数获得期望的数据。枚举参数将期望值分为三类，分别是字符串、整型或者双精度浮点数，对应的枚举为`CURLResponse.Info.StringValue`、`CURLResponse.Info.IntValue`和`CURLResponse.Info.DoubleValue`。此外，`get`函数也可以用于直接从响应头提取数据：

``` swift
public extension CURLResponse {
	/// 从响应数据中提取字符串
	func get(_ stringValue: Info.StringValue) -> String?
	/// 将响应数据视为整型值
	func get(_ intValue: Info.IntValue) -> Int?
	/// 将响应数据视为双精度浮点数
	func get(_ doubleValue: Info.DoubleValue) -> Double?
	/// 获取响应头数据值，返回第一个值的实例；如果没有实例则返回空。
	func get(_ header: Header.Name) -> String?
	/// 获取响应头数据值，返回所有实例并组成字符串数组
	func get(all header: Header.Name) -> [String]
}
```

另外该对象还增加了一些便捷属性用于从响应中提取数据，如`url`和`responseCode`状态值。

下列源码展示了如何从请求中提取头数据或者其他元数据：

``` swift
// 获取响应代码
let code = response.get(.responseCode)
```

```swift
// 直接调用属性的方法获取响应代码
let code = response.responseCode
```

```swift
// 从响应中获取“Last-Modified”（最近修改）这个变量的值
if let lastMod = response.get(.lastModified) {
	...
}
```

### 访问失败

如果访问失败的情况发生，则程序会抛出一个 `CURLResponse.Error` 对象。该对象包含CURL错误代码（注意是CURL本身的错误代码，不是HTTP响应错误代码），同时还提供一个错误消息解释；对应的CURLResponse对象涵盖了这个错误，便于查询。

请注意，根据请求对象选项具体设置，在出错情况下的响应对象有可能不包含任何响应数据内容。

`CURLResponse.Error` 对象定义如下：

```swift
open class CURLResponse {
	/// 执行网络访问时可能会发生的错误
	public struct Error: Swift.Error {
		/// curl 错误代码
		public let code: Int
		/// curl 错误代码对应的消息解释
		public let description: String
		/// 产生错误的响应对象
		public let response: CURLResponse
	}
}
```


### CURLRequest.Option 选项清单

以下是`CURLRequest`对象可以设置的选项清单；每一个枚举都对应了该选项的参数类型。这些枚举值可用于创建一个新的`CURLRequest`对象，或者追加到现有的`.options`数组属性中去。

|CURLRequest.Option 枚举类型|描述|
|---|---|
|.url(String)|请求的目标URL地址|
|.port(Int)|请求的目标端口|
|.failOnError|如果网络HTTP响应代码超过400时报错|
|.userPwd(String)|用户名密码字符串，中间用冒号分割|
|.proxy(String)|代理服务器地址|
|.proxyUserPwd(String)|代理服务器用户名/密码组合|
|.proxyPort(Int)|代理服务器地址|
|.timeout(Int)|请求超时设置，单位是秒。默认永不超时|
|.connectTimeout(Int)|请求发起时等待连接的最长时间，单位时秒。默认为300秒|
|.lowSpeedLimit(Int)|低速限制。如果平均每秒传输字节总量低于该门槛值，则视为网络连接处于低速状态|
|.lowSpeedTime(Int)|如果低速状态持续时间超过了该门槛值限制的时间（单位是秒）则视为连接过慢，需要放弃、|
|.range(String)|以“X-Y”格式声明的范围字符串，其中X或Y任何一个范围限制都可以开口（X缺省为0而Y缺省为获得所有数据），否则X和Y的值为访问返回的数据体字节索引范围（译者注：比如假设返回结果为Hello，则范围1-3返回的内容为ell）|
|.resumeFrom(Int)|如果设置了本选项，则请求将从本选项指定的字节偏移量开始工作|
|.cookie(String)|为请求设置一个或多个cookie，每个曲奇表达式都必须以“name=value”作为格式。多赋值曲奇🍪应该以分号分割，比如这样：“name1=value1; name2=value2”|
|.cookieFile(String)|提示系统曲奇数据来自于本地文件；参数为本地文件名|
|.cookieJar(String)|用于接收返回数据的曲奇文件名称，即收到曲奇文件后数据会写到这个本地文件中去|
|.followLocation(Bool)|提示请求是否要根据响应返回的地址重新定向到新的地址上去；默认为 i假，即收到了重定向地址不会自动跳转|
|.maxRedirects(Int)|最大跳转数量；如果允许地址重定向，则允许的最大地址重定向次数。默认是不限次数|
|.maxConnects(Int)|最大连接数，即允许为同一请求缓冲的最大TCP同步连接数量|
|.autoReferer(Bool)|自动参考。如果设置为真，则请求会在自动跳转发生时，自动设置HTTP头数据中Referer变量参考地址用于表明跳转来源。|
|.krbLevel(KBRLevel)|为FTP文件传输协议设置Kerberos安全等级，允许值为：.clear（清除）、.safe（安全）、.confidential（凭证）或 .private（私有）|
|.addHeader(Header.Name, String)|在请求中增加一个头数据变量|
|.addHeaders([(Header.Name, String)])|在请求中增加一系列头数据变量|
|.replaceHeader(Header.Name, String)|增加或者替换某个头数据变量|
|.removeHeader(Header.Name)|删除某个默认的头数据变量|
|.sslCert(String)|指向客户端的SSL证书地址路径|
|.sslCertType(SSLFileType)|设置客户端SSL证书类型，默认为`.pem`|
|.sslKey(String)|客户端证书钥匙文件|
|.sslKeyPwd(String)|如果客户端证书有密码保护，则请在此书输入证书密码|
|.sslKeyType(SSLFileType)|说明SSL钥匙文件的类型|
|.sslVersion(TLSMethod)|强制请求使用特定类型的TLS或者SSL|
|.sslVerifyPeer(Bool)|说明证书是否使用双向认证。|
|.sslVerifyHost(Bool)|说明服务器证书是否必须在已知主机列表中列出|
|.sslCAFilePath(String)|此处声明用于双向认证的本地证书文件路径|
|.sslCADirPath(String)|此处声明用于双向认证的本地证书集合所在的文件夹|
|.sslCiphers([String])|覆盖用于SSL链接的密码列表|
|.sslPinnedPublicKey(String)|固化的公共钥匙文件路径。当初始化链接进行TLS/SSL协商时，服务器会发过来一个证书用于说明其身份；从该证书中可以分离出来一个公有钥匙，如果被分离出来的公有钥匙与选项中提供的钥匙不吻合，则curl会中断链接，停止发送或接收任何数据。|
|.ftpPreCommands([String])|在发送文件之前需要执行的FTP命令清单|
|.ftpPostCommands([String])|在发送文件之后需要执行的FTP命令清单|
|.ftpPort(String)|明确用于执行FTP请求的本地连接端口号|
|.ftpResponseTimeout(Int)|执行FTP请求时允许等待的最长时间，单位是秒|
|.sshPublicKey(String)|用于SSH链接的公有钥匙文件路径|
|.sshPrivateKey(String)|用于SSH链接的私有钥匙文件路径|
|.httpMethod(HTTPMethod)|用于请求的HTTP方法|
|.postField(POSTField)|增加一个POST表单数据；通常一个请求中都允许增加多个POST表单字段|
|.postData([UInt8])|用于POST请求表单的二进制字节数据|
|.postString(String)|用于POST表单数据的字符串表达|
|.mailFrom(String)|用于执行SMTP简单邮件协议请求的邮件发送方地址|
|.mailRcpt(String)|用于执行SMTP简单邮件协议请求的邮件接收方地址，可以多次使用（用于抄送和转发）|

### CURLResponse.Info 清单

以下清单用于描述`CURLResponse.Info`的枚举类型，便于`CURLResponse.get`函数获取响应数据。每一组列表都对应着不同的数据类型，分别是`StringValue`（字符串值）、`IntValue`（整型值）和`DoubleValue`（双精度浮点数类型）。

|CURLResponse.Info.StringValue 枚举类型|描述说明|
|---|---|
|.url|用于请求／响应的有效URL地址。以最终返回的响应数据中包括的URL地址为准，可能与最初请求时发出的地址有所不同，比如响应时出现重定向和地址跳转|
|.ftpEntryPath|登录到FTP服务器后的默认路径|
|.redirectURL|⚠️请求发起之后⚠️应该跳转到的URL地址|
|.localIP|最近一次本地使用的IP地址|
|.primaryIP|最近一次连接到的远程服务器所使用的地址|
|.contentType|请求的内容类型，从头数据的“Content-Type”变量中读出|

|CURLResponse.Info.IntValue 枚举类型|描述说明|
|---|---|
|.responseCode|响应代码。即最新获得的HTTP／FTP或者SMTP响应时发出的代码|
|.headerSize|所有已收到的头数据总长度，单位是字节|
|.requestSize|所有已发出的请求头数据总长度，单位是字节。如果出现了重定向则该长度会根据连续跳转而导致的头数据内容增加的情况，该长度会随之积累|
|.sslVerifyResult|SSL证书验证结果|
|.redirectCount|重定向跳转总次数|
|.httpConnectCode|最近收到的HTTP代理服务器响应代码|
|.osErrno|如果出现错误，则当前操作系统级别的错误代码|
|.numConnects|用于完成该响应的TCP链接总数|
|.primaryPort|服务器对方目前最近一次连接到的远程端口号|
|.localPort|最近一次请求使用的本地TCP端口号|

|CURLResponse.Info.DoubleValue 枚举类型|描述说明|
|---|---|
|.totalTime|最后一次请求总共耗时，单位是秒|
|.nameLookupTime|域名解析所用时间，单位是秒|
|.connectTime|连接到远程服务器或者代理服务器所用时间，单位是秒|
|.preTransferTime|文件传输开始之前所耗时间，单位是秒|
|.sizeUpload|已上传数据总字节数|
|.sizeDownload|已下载数据总字节数|
|.speedDownload|平均下载速度，单位是字节/秒|
|.speedUpload|平均上传速度，单位是字节/秒|
|.contentLengthDownload|下载内容长度，单位是字节。该数据值是从头数据的“Content-Length”中获取|
|.contentLengthUpload|上传内容长度，单位是字节|
|.startTransferTime|从请求开始到下载第一个字节时所耗时间，单位是秒|
|.redirectTime|重定向所耗时间，单位是秒；重定向耗时包括以下各个阶段所耗时间总和：域名解析、TCP连接、传输开始之前所耗时间、在最终会话开始之前的传输耗时|
|.appConnectTime|应用连接时间，单位是秒。该时间从连接开始计时，截止至SSL／SSH握手结束|



