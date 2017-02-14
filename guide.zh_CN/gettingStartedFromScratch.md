# 从模板项目开始

本章将引导您使用Swift和Perfect软件框架逐步设置一个简单的HTTP服务器。

## 系统要求

### Swift 3.0

在您从[Swift.org（英文版）](https://swift.org/getting-started/)完成Swift 3.0 toolchain工具集安装之后，请打开一个命令行终端并输入命令
```
swift --version
```

如果能够看到类似于下面的消息就对了：

```
Apple Swift version 3.0 (swiftlang-800.0.33.1 clang-800.0.31)
Target: x86_64-apple-macosx10.9
```
请注意您需要最新版本的Swift 3.0，无论是预览版（preview）还是快照版（snapshot）。如果低于3.0版本则Perfect是无法成功编译的。

您可以从以下连接查看您当前的项目究竟需要哪一个Swift版本：[Perfect主代码资源库的说明文件（英文版）](https://github.com/PerfectlySoft/Perfect#compatibility-with-swift)

### OS X
您需要的所有内容均已预装。

### Ubuntu Linux
Perfect软件框架可以在Ubuntu Linux 14.04 and 15.10环境下运行。Perfect依赖于若干软件接口库，比如OpenSSL、libssl-dev和uuid-dev。如果需要安装这些内容，请在终端控制台内输入：

```
sudo apt-get install openssl libssl-dev uuid-dev
```

### 创建Swift软件包

注意现在您已经可以准备开始建议一个Web应用了。请新建一个文件夹用于保存项目文件：

```
mkdir MyAwesomeProject
cd MyAwesomeProject
```

为了加快项目进度，最简单的方法就是把这个项目目录转化为git repo（代码资源文件夹）：

```
git init
touch README.md
git add README.md
git commit -m "Initial commit"
```

另外还推荐您参考[Swift的一个.gitignore文件模板（来源于Gitignore.io）](https://www.gitignore.io/api/swift)内容增加一个`.gitignore`文件（用于通知git资源库在本文件中列出的文件和文件夹都是本地临时文件，不需要上传到共享的git资源库）。

现在请在您的git repo根目录下面创建一个`Package.swift`文件。这个文件是SPM（Swift软件包管理器）编译项目时必须要用到的文件。

``` swift
import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        .Package(
        url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
        majorVersion: 2, minor: 0
        )
    ]
)
```

接下来请创建一个名为`Sources`的文件夹用于保存源程序，然后在这个源程序文件夹下面创建一个`main.swift`文件，内容如下：

```
mkdir Sources
echo 'print("您好！")' >> Sources/main.swift
```

现在项目就已经准备好，可以通过以下两个命令编译和运行：

```
swift build
.build/debug/MyAwesomeProject
```

*⚠️注意⚠️* 如果编译过程中有问题，那么请尝试用特定的一个工具集来编译，形式如下：
`/Library/Developer/Toolchains/swift-DEVELOPMENT-SNAPSHOT-2016-08-18-a.xctoolchain/usr/bin/swift build`

编译完成后运行，应该看到：

```
你好！！！
```

### 设置服务器

现在已经确认Swift工具包已经准备好了。下一步就可以实现一个Perfect的HTTPServer服务器啦！打开`Sources/main.swift`文件，把内容替换为以下程序：

``` swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

// 创建HTTP服务器
let server = HTTPServer()

// 注册您自己的路由和请求／响应句柄
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
        request, response in
        response.setHeader(.contentType, value: "text/html")
        response.appendBody(string: "<html><title>你好，世界！</title><body>你好，世界！</body></html>")
        response.completed()
    }
)

// 将路由注册到服务器上
server.addRoutes(routes)

// 监听8181端口
server.serverPort = 8181

do {
    // 启动HTTP服务器
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("网络出现错误：\(err) \(msg)")
}
```

然后再次编译运行当前项目：

```
swift build
.build/debug/MyAwesomeProject
```

此时服务器就运行了，随时等待网络连接！🎉 从浏览器打开[http://localhost:8181/](http://127.0.0.1:8181/)就可以看到欢迎信息。在终端控制台上用组合键“control-c”可以随时停止服务器。

### Xcode

Swift软件包管理器（SPM）能够创建一个Xcode项目，并且能够运行PerfectTemplate模板服务器，还能为您的项目提供完全的源代码编辑和调试。在您的终端命令行内输入：

```
swift package generate-xcodeproj
```

然后打开产生的文件“PerfectTemplate.xcodeproj”，在”Library Search Paths“检索项目软件库中增加（不单单是编译目标）：

```
$(PROJECT_DIR) - Recursive
```

确定选择了可执行的目标文件，并选择在“我的Mac”运行。同样注意要选择正确的Swift toolchain工具集。现在您就可以在Xcode中运行调试您的服务器了！
