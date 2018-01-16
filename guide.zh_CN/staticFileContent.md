# 静态文件

参考[请求响应路由](routing.md)一章，Perfect可以实现非常复杂的请求响应路由与定向，也因此当然可以很好地管理如HTML、图片、CSS样式表和JavaScript脚本这样的静态文件内容。

静态文件内容的调用是通过```StaticFileHandler```对象完成。一旦该对象实例收到请求，该对象会去检索目标文件，如果没有找到就返回404页面“文件不存在”。一个```StaticFileHandler```对象还可以通过使用ETag文件头的方法来缓冲处理非常大的文件。

当```StaticFileHandler```对象初始化时需要一个文档根目录作为参数。该根目录将成为其它所有静态文件的路径前缀。当前的HTTPRequest对象实例中的```path```路径属性将会用于显示目标文件的具体路径。

```StaticFileHandler```可以从您的网站句柄直接调用```handleRequest```方法而来

比如，句柄可以用于根据请求直接返回响应，如下所示：

``` swift
{
    request, response in
    StaticFileHandler(documentRoot: request.documentRoot)
		.handleRequest(request: request, response: response)
}
```

如果您的服务器只存放静态网页，则可以直接从文档根目录启动：

``` swift
try HTTPServer.launch(.server(name: "localhost", port: 8080, documentRoot: "/path/to/webroot"))
```

以下示例展示了一个虚拟的文档路径，所有"/files"开头的URI都映射到了物理路径"/var/www/htdocs"


``` swift
routes.add(method: .get, uri: "/files/**") {
    request, response in

    // 获得符合通配符的请求路径
    request.path = request.urlVariables[routeTrailingWildcardKey]

    // 用文档根目录初始化静态文件句柄
    let handler = StaticFileHandler(documentRoot: "/var/www/htdocs")

    // 用我们的根目录和路径
    // 修改集触发请求的句柄
    handler.handleRequest(request: request, response: response)
)
```

在之前的请求响应路由例子中，一个发向“/files/foo.html”的请求将返回对应的文件“/var/www/htdocs/foo.html”

### 页面压缩和性能提升

默认情况下，StaticFileHandler 静态页面处理器会全力以赴把所有文件内容推送给客户端，也就是使用套接字的 `sendfile` 方法。不过有两种情况是无法实现的或者不希望发生的。

第一种情况是是用TLS/SSL 加密通信。原本文件处理器会把整个文件的打开并发出所有数据块，而一旦采取加密形式，文件处理器不会有额外操作，只不过检测到加密连接时会关闭 `sendfile`的使用。

另一种情况是流量压缩。内容压缩的过程不会和 `sendfile`进行混合使用。如果要启动压缩功能，只需要把`allowResponseFilters`打开即可：

```swift
StaticFileHandler(documentRoot: "/path/to/root/", allowResponseFilters: true)
```