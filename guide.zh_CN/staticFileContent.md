# 静态文件

参考[请求响应路由](routing.md)一章，Perfect可以实现非常复杂的请求响应路由与定向，也因此当然可以很好地管理如HTML、图片、CSS样式表和JavaScript脚本这样的静态文件内容。

静态文件内容的调用是通过```StaticFileHandler```对象完成。一旦该对象实例收到请求，该对象会去检索目标文件，如果没有找到就返回404页面“文件不存在”。一个```StaticFileHandler```对象还可以通过使用ETag文件头的方法来缓冲处理非常大的文件。

当```StaticFileHandler```对象初始化时需要一个文档根目录作为参数。该根目录将成为其它所有静态文件的路径前缀。当前的HTTPRequest对象实例中的```path```路径属性将会用于显示目标文件的具体路径。

```StaticFileHandler```可以从您的网站句柄直接调用```handleRequest```方法而来

比如，句柄可以用于根据请求直接返回响应，如下所示：

``` swift
{
    request, response in
    StaticFileHandler(documentRoot: request.documentRoot).handleRequest(request: request, response: response)
}
```

但实际上没必要这么做，除非您需要定制静态文件句柄对象的行为。只要设置服务器的`documentRoot`属性就能够根据目标目录自动安装一个能够服务多个文件的句柄。服务器设置类似与以下代码：

``` swift
let dir = Dir(documentRoot)
if !dir.exists {
    try Dir(documentRoot).create()
}
routes.add(method: .get, uri: "/**", handler: {
    request, response in
    StaticFileHandler(documentRoot: request.documentRoot).handleRequest(request: request, response: response)
})
```

该句柄安装后可以服务于任何时候文档根目录下的文件以及包含的各级子目录。

documentRoot属性的例子可以在PerfectTemplate项目模板中找到：

``` swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

// 创建HTTP服务
let server = HTTPServer()

// 注册您自己的网页路由和控制句柄
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
        request, response in
        response.appendBody(string: "<html>...</html>")
        response.completed()
    }
)

// 将路由增加到服务器上
server.addRoutes(routes)

// 设置监听端口为8181
server.serverPort = 8181

// 设置文档根目录
// 这是可选的。
// 如果不希望提供静态内容就不需要设置。
// 设置文档根目录后，
// 系统会自动为路由增加一个静态文件处理句柄
server.documentRoot = "./webroot"

// 从命令行获得选项并继续配置服务器
// 用运行服务器的命令行增加--help来浏览所有支持的参数。
// 命令行参数会代替以上任何一个配置值。
configureServer(server)

do {
    // 启动HTTP服务器
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("网络发生错误：\(err) \(msg)")
}

```

*⚠️注意⚠️* `server.documentRoot = "./webroot"`这一行。这意味着如果这里有一个styles.css文件，则任何关于URL"/styles.css"的请求都会将这个style.css文件返回给浏览器

下面的例子建立了一个虚拟的文档目录，向所有资源路径"/files"开头的URL从物理目录"/var/www/htdocs"提供服务并响应。

``` swift
routes.add(method: .get, uri: "/files/**", handler: {
    request, response in

    // 获得符合通配符的请求路径
    request.path = request.urlVariables[routeTrailingWildcardKey]!

    // 用文档根目录初始化静态文件句柄
    let handler = StaticFileHandler(documentRoot: "/var/www/htdocs")

    // 用我们的根目录和路径
    // 修改集触发请求的句柄
    handler.handleRequest(request: request, response: response)
    }
)
```

在之前的请求响应路由例子中，一个发向“/files/foo.html”的请求将返回对应的文件“/var/www/htdocs/foo.html”
