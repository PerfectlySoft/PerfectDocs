# 从模板项目开始

本章将引导您使用Swift和Perfect软件框架逐步设置一个简单的HTTP服务器。

## 系统要求

### Swift 4.0

在您从[Swift.org（英文版）](https://swift.org/getting-started/)完成Swift 4.0 toolchain工具集安装之后，请打开一个命令行终端并输入命令
```
swift --version
```

如果能够看到类似于下面的消息就对了：

```
Apple Swift version 3.0 4.0 (swiftlang-900.0.65 clang-900.0.37)
Target: x86_64-apple-macosx10.9
```
请注意您需要最新版本的Swift 4.0，无论是预览版（preview）还是快照版（snapshot）。如果低于4.0版本则Perfect是无法成功编译的。

您可以从以下连接查看您当前的项目究竟需要哪一个Swift版本：[Perfect主代码资源库的说明文件（英文版）](https://github.com/PerfectlySoft/Perfect#compatibility-with-swift)

### macOS
您需要的所有内容均已预装。

### Ubuntu Linux
Perfect软件框架可以在Ubuntu Linux 16.04 环境下运行。Perfect依赖于若干软件接口库，比如OpenSSL、libssl-dev和uuid-dev。如果需要安装这些内容，请在终端控制台内输入：

```
sudo apt-get install openssl libssl-dev uuid-dev
```

### 创建Swift软件包

注意现在您已经可以准备开始建议一个Web应用了。请新建一个文件夹用于保存项目文件：

```
mkdir MyAwesomeProject
cd MyAwesomeProject
```

用SPM软件包管理器初始化项目：

```
swift package init --type=executable
```
这个命令会在当前工作目录自动生成下列文件：

```
Creating executable package: MyAwesomeProject
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/MyAwesomeProject/main.swift
Creating Tests/
```

另外还推荐您参考[Swift的一个.gitignore文件模板（来源于Gitignore.io）](https://www.gitignore.io/api/swift)内容增加一个`.gitignore`文件（用于通知git资源库在本文件中列出的文件和文件夹都是本地临时文件，不需要上传到共享的git资源库）。

然后打开Package.swift文件进行编辑：

``` swift
// swift-tools-version:4.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
            name: "MyAwesomeProject",
            dependencies: []),
    ]
)
```
然后修改以上内容为：

``` swift
// swift-tools-version:4.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        .package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", from: "3.0.0")
    ],
    targets: [
        .target(
            name: "MyAwesomeProject",
            dependencies: ["PerfectHTTPServer"]),
    ]
)
```

可以注意到此时该文件在要求Swift编译的最低版本为3.0.0，同样需要在目标编译生成文件增加PerfectHTTPServer的依存关系。

接下来请需要修改`Sources/MyAwesomeProject/main.swift`内容：

```
mkdir Sources
echo 'print("您好！")' >> Sources/MyAwesomeProject/main.swift
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

现在已经确认Swift工具包已经准备好了。下一步就可以实现一个Perfect的HTTPServer服务器啦！打开`Sources/MyAwesomeProject/main.swift`文件，把内容替换为以下程序：

``` swift
import PerfectHTTP
import PerfectHTTPServer

// 注册您自己的路由和请求／响应句柄
var routes = Routes()
routes.add(method: .get, uri: "/") {
	request, response in
	response.setHeader(.contentType, value: "text/html")
	response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
		.completed()
}

do {
    // 启动HTTP服务器
    try HTTPServer.launch(
		.server(name: "www.example.ca", port: 8181, routes: routes))
} catch {
	fatalError("\(error)") // fatal error launching one of the servers
}
```

然后再次编译运行当前项目：

```
swift build
.build/debug/MyAwesomeProject
```

应该能看到服务器在启动：

```
[INFO] Starting HTTP server www.example.ca on :::8181
```


此时服务器就运行了，随时等待网络连接！🎉 从浏览器打开[http://localhost:8181/](http://127.0.0.1:8181/)就可以看到欢迎信息。在终端控制台上用组合键“control-c”可以随时停止服务器。

### Xcode

Swift软件包管理器（SPM）能够为您的项目MyAwesomeProject生成Xcode工程：

```
swift package generate-xcodeproj
```

然后打开产生的文件“MyAwesomeProject”，在”Library Search Paths“检索项目软件库中增加（不单单是编译目标）：

```
$(PROJECT_DIR) - Recursive
```


在Xcode打开项目之后，请注意选择可执行目标为 "My Mac"，并选择正确的Swift工具链。为了使服务器能够访问您工程文件夹下的目录，比如静态网页，请选择菜单命令 "Product > Scheme > Edit Scheme…" 然后设置工作目录 "Use Custom Working Directory" 为项目文件夹。 现在您就可以在Xcode中运行调试您的服务器了
