# 快速上手

您期望用Perfect和Swift进行编程吗？本文将向您介绍所有需要运行Perfect的基本内容，并帮助您创建第一个应用程序。

阅读本文后，您会了解到：

- 如何创建和运行一个HTTP/HTTPS服务器，将Perfect运转起来
- 安装运行Perfect之前，无论是OS X还是Ubuntu Linux系统都需要安装的必要软件
- 如何为Swift项目编译、测试和管理依存关系
- 如何将Perfect部署到其它环境中，比如Heroku、Amazon Web Services、Docker、Microsoft Azure、Google Cloud、IBM Bluemix CloudFoundry，以及IBM Bluemix Docker

### Swift 4.0

在您从[Swift.org（英文版）](https://swift.org/getting-started/)完成Swift 4.0 toolchain工具集安装之后，请打开一个命令行终端并输入命令
```
swift --version
```

如果能够看到类似于下面的消息就对了：

```
Apple Swift version 4.0 (swiftlang-900.0.65 clang-900.0.37)
Target: x86_64-apple-macosx10.9
```
请注意您需要最新版本的Swift 4.0。如果低于4.0版本则Perfect是无法成功编译的。

### Ubuntu Linux
Perfect软件框架可以在Ubuntu Linux 16.04 环境下运行。Perfect依赖于若干软件接口库，比如OpenSSL、libssl-dev和uuid-dev。如果需要安装这些内容，请在终端控制台内输入：

```
sudo apt-get install openssl libssl-dev uuid-dev
mac os x命令更换为：brew install openssl
```

### Perfect项目快速上手

现在一切准备就绪，您可以开始开发您的第一个Web应用入门项目。

### 编译入门项目

执行以下命令能够克隆并编译一个空的入门项目。编译后可以启动一个本地的服务器，监听您计算机的8181端口：

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
.build/debug/PerfectTemplate
```

您应该可以在终端控制台中看到类似下面的内容：

```
[INFO] Starting HTTP server localhost on 0.0.0.0:8181
```

服务器现在已经运行并等待连接。从浏览器打开[http://localhost:8181/](http://127.0.0.1:8181/) 可以看到欢迎信息。在终端控制台中输入组合键“control-c”可以随时终止服务器运行。

完整的源代码请参考[PerfectTemplate项目模板](https://github.com/PerfectlySoft/PerfectTemplate)。

### Xcode

Swift软件包管理器（SPM）能够创建一个Xcode项目，并且能够运行PerfectTemplate模板服务器，还能为您的项目提供完全的源代码编辑和调试。在您的终端命令行内输入：

```
swift package generate-xcodeproj
```

然后打开产生的文件“PerfectTemplate.xcodeproj”，确定选择了可执行的目标文件，并选择在“我的Mac”运行。现在您可以运行并调试服务器了。

## 下一步

以下的源代码例子显示了几种在开发web应用或者REST中应用常见的任务。在所有的例子中，```request```请求和```response```响应变量分别代表了传递给您的请求／响应句柄的```HTTPRequest```请求和```HTTPResponse```响应对象。

上述对象详见[API参考手册](https://perfect.org/docs/api.html)

### 获得客户端请求消息头

``` swift
if let acceptEncoding = request.header(.acceptEncoding) {
	...
}
```

### 获得客户端GET和POST参数

``` swift
if let foo = request.param(name: "foo") {
	...
}
if let foo = request.param(name: "foo", defaultValue: "default foo") {
	...
}
let foos: [String] = request.params(named: "foo")
```

### 获得当前的请求路径

``` swift
let path = request.path
```

### 访问服务器文档目录并返回给客户端一个图像文件

``` swift
let docRoot = request.documentRoot
do {
    let mrPebbles = File("\(docRoot)/mr_pebbles.jpg")
    let imageSize = mrPebbles.size
    let imageBytes = try mrPebbles.readSomeBytes(count: imageSize)
    response.setHeader(.contentType, value: MimeType.forExtension("jpg"))
    response.setHeader(.contentLength, value: "\(imageBytes.count)")
    response.setBody(bytes: imageBytes)
} catch {
    response.status = .internalServerError
    response.setBody(string: "请求处理出现错误： \(error)")
}
response.completed()
```

### 获得客户端浏览器内的Cookie

``` swift
for (cookieName, cookieValue) in request.cookies {
	...
}
```

### 设置客户端浏览器内的Cookie

``` swift
let cookie = HTTPCookie(name: "cookie-name", value: "the value", domain: nil,
                    expires: .session, path: "/",
                    secure: false, httpOnly: false)
response.addCookie(cookie)
```

### 将JSON数据返回给客户端

``` swift
response.setHeader(.contentType, value: "application/json")
let d: [String:Any] = ["a":1, "b":0.1, "c": true, "d":[2, 4, 5, 7, 8]]

do {
    try response.setBody(json: d)
} catch {
    //...
}
response.completed()
```
*上述代码使用了内建的JSON编码功能，请根据需要自行使用JSON编解码功能*

### 客户端请求重定向

``` swift
response.status = .movedPermanently
response.setHeader(.location, value: "http://www.perfect.org/")
response.completed()
```

### 以定制风格过滤和处理404错误

``` swift
struct Filter404: HTTPResponseFilter {
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}

	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		if case .notFound = response.status {
			response.bodyBytes.removeAll()
			response.setBody(string: "The file \(response.request.path) was not found.")
			response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
			callback(.done)
		} else {
			callback(.continue)
		}
	}
}

try HTTPServer(documentRoot: webRoot)
	.setResponseFilters([(Filter404(), .high)])
	.start(port: 8181)
```
