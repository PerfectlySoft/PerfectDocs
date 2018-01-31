# 用SPM软件包管理器编译项目
[Swift软件包管理器（SPM）](https://swift.org/package-manager/) 是一个用于Swift项目开发、测试、依存关系管理的命令行工具。所有的Perfect组件针对SPM设计的。如果您需要使用Perfect进行项目开发，同样需要使用SPM。

创建一个新的Perfect项目最好的办法就是克隆[PerfectTemplate项目模板](https://github.com/PerfectlySoft/PerfectTemplate)或从该模板上开发新的Perfect分支。该模板提供一个非常简单的服务器并返回一条“你好，世界！”的服务器消息。您可以根据需要修改编辑。

在开始之前，请详细阅读[依存关系](https://github.com/PerfectlySoft/Perfect/wiki/Dependencies)文档，同时您需要在您的系统平台上安装Swift 4.0 toolchain工具集。

下一步是克隆项目模板。请从控制台命令行改变到您希望克隆项目的具体路径。将下列命令输入终端控制台然后下载名为“PerfectTemplate”的项目模板：

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
```

在Perfect Template项目模板中，您可以找到两个很重要的文件：

- 其中 *Sources* 目录包含了所有Perfect项目的Swift源程序文件
- 还有一个名为“*Package.swift*”的SPM文件管理清单，包含了整个项目对其它库函数的依存关系

所有的SPM项目至少要包括一个 *Sources* 目录和一个 *Package.swift* 文件。而项目模板中目前只有一个依存关系：[Perfect-HTTPServer服务器](https://github.com/PerfectlySoft/Perfect-HTTPServer)项目

### 在Package.swift文件中的依存

PerfectTemplate项目模板*Package.swift*清单文件包含了以下内容：

``` swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
			majorVersion: 3)
    ]
)
```

**⚠️注意⚠️** 例子中的版本信息可能与您最终从终端控制台获取的版本有出入。我们建议您从实际的代码库中获取最新的内容。

在 *Package.swift* 文件中由两个重要的内容您可能需要编辑。

第一个是 **name** 项目名称，用于说明当前项目的目标名称，因此最后可执行文件的名字也会按照这个名称进行编译。

第二个是 **dependencies** 依存关系清单。该内容说明了您的应用程序需要的所有子项目列表，在这个数组中其中每一个条目都包含了一个“*.Package*”软件包，及其来源URL和版本。

上面的例子用一个宽范围的版本来保证模板项目依赖于HTTPServer服务器项目最新的版本。您可能希望限制当前项目于一个较为稳定的版本上。比如，如果只希望用v2版本的Perfect HTTPServer项目，则您的“*.Package*”条目可能看起来像下面这样：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", majorVersion: 3)
```

相信您的项目会不断发展，因此您也需要不断增加更多的库函数依存关系，所以需要将所有这些软件包的内容添加到**dependencies**清单中去。SPM会自动下载适合的版本配合您的项目一同编译。所有依存关系库文件都会下载到一个SPM自动创建的*Packages*目录下。比如，如果您希望在您的服务器上使用Mustache模板，则您的*Package.swift*文件可能会看起来像这样：

``` swift
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
			majorVersion: 3),
		.Package(url: "https://github.com/PerfectlySoft/Perfect-Mustache.git",
			majorVersion: 3)
    ]
)
```

通过这种方式[Perfect-Mustache](https://github.com/PerfectlySoft/Perfect-Mustache)项目就变成了您当前工程的组成部分，为您的Perfect服务器使用Mustache模板提供支持。现在您在您的Swift代码中可以声明`import PerfectMustache`并使用该库函数所带的各种功能。

伴随着您的工程依存关系逐渐增多，您可能希望以另外一种方式进行管理。以下项目包含了所有Perfect代码资源库，每个URL链接都是分开维护的，并且被映射到 **dependencies** 依存关系参数所需的格式。

``` swift
import PackageDescription

let versions = Version(0,0,0)..<Version(10,0,0)
let urls = [
    "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
    "https://github.com/PerfectlySoft/Perfect-FastCGI.git",
    "https://github.com/PerfectlySoft/Perfect-CURL.git",
    "https://github.com/PerfectlySoft/Perfect-PostgreSQL.git",
    "https://github.com/PerfectlySoft/Perfect-SQLite.git",
    "https://github.com/PerfectlySoft/Perfect-Redis.git",
    "https://github.com/PerfectlySoft/Perfect-MySQL.git",
    "https://github.com/PerfectlySoft/Perfect-MongoDB.git",
    "https://github.com/PerfectlySoft/Perfect-WebSockets.git",
    "https://github.com/PerfectlySoft/Perfect-Notifications.git",
    "https://github.com/PerfectlySoft/Perfect-Mustache.git"
]

let package = Package(
    name: "PerfectTemplate",
    targets: [],
    dependencies: urls.map { .Package(url: $0, versions: versions) }
)
```

### 编译

SPM提供以下命令用于编译项目，并且清理旧的编译结果：

```
swift build
```

该命令能够自动下载需要的依存文件，如果全部依存关系都已经成功获取到本地计算机，则开始项目编译工作。如果编译成功，则可执行文件会被放到一个隐蔽目录`.build/debug/`。当编译PerfectTemplate模板项目时，您会看到SPM输出的最后一句话：`Linking .build/debug/PerfectTemplate`。在终端控制台输入`.build/debug/PerfectTemplate`即可启动服务器。默认情况下编译出的是一个用于调试的版本。如果需要编译一个用于发行的版本，请用命令`swift build -c release`。运行后可发行版本的可执行程序被放在了`.build/release/`目录下。

```
swift build --clean
```

```
swift build --clean=dist
```

上述命令能够清理所有编译临时文件并产生一个干净的版本。只要使用`--clean`选项，SPM就会删除`.build`，然后重新开始一个全新的编译。如果使用的命令行包含`--clean=dist`选项， 则`.build`目录和`Packages`目录都会被删除。用这个命令能够重新下载所有依存关系以获得最新版本对项目的支持。

### Xcode项目

SPM还能根据您的 *Package.swift* 文件创建对应的Xcode项目。该项目允许您使用Xcode编译和调试。要生成Xcode项目，请使用以下命令行：

```
swift package generate-xcodeproj
```

该命令会将Xcode项目建立在同一目录下。比如，在PerfectTemplate项目根目录下调用该命令会得到一个终端控制台消息：`generated: ./PerfectTemplate.xcodeproj`，即Xcode项目已经生成。

**⚠️注意⚠️**：最好不要在这个Xcode项目上直接编辑或增加文件。如果需要更多的依存关系，或者需要下载更新的版本，您需要重新生成这个Xcode项目。因此，在之前您做的任何修改都会被Xcode覆盖。

### 更多信息

关于SPM的更多详细信息，请访问：

[Swift软件包管理器（英文版）](https://swift.org/package-manager/)
