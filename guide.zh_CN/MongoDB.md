# MongoDB
MongoDB库函数是在mongo-c语言库的基础上封装而成，能够为Swift轻松访问MongoDB服务器提供便利。

该工具库软件包是由Swift软件包管理器编译而来，是
[Perfect](https://github.com/PerfectlySoft/Perfect)项目的组成部分，
被设计为可以独立使用，不依赖PerfectLib或其它任何组件。

请确保安装并激活了最新版本的Swift 4.0 toolchain。

### 不同操作系统平台的准备工作

### OS X

该工具包需要通过[Homebrew](http://brew.sh/)安装mongo-c。

安装Homebrew：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装mongo-c:

```
brew install mongo-c-driver
```

### Linux

确保已经安装了libmongoc。

```
sudo apt-get install libmongoc
```

### 在您的项目里引用MongoDB Driver驱动

请在Package.swift增加对该驱动的依存关系。

``` swift
.Package(
	url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git",
	majorVersion: 3
	)
```

关于如何在您的项目中使用Perfect函数库，详见参考手册《[使用Swift软件包管理器编译项目](buildingWithSPM.md)》

### 快速上手

通过以下命令快速克隆一个空白的Perfect项目模板：

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
```

在Package.swift文件中增加依存关系：

```swift
let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url:"https://github.com/PerfectlySoft/Perfect.git", versions: Version(0,0,0)..<Version(10,0,0)),
		.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
	]
)
```

创建Xcode项目：

```
swift package generate-xcodeproj
```

从Xcode中打开自动生成的`PerfectTemplate.xcodeproj`项目文件。

该项目会编译然后在本地端口8181启动一个服务器。

> **⚠️注意⚠️** 每次向项目追加依存关系时，必须要打开Swift软件包管理器重新创建一个新的Xcode项目文件。注意任何对该文件的手工修改都会被丢弃。

### 在您的项目中声明MongoDB

请在您的Perfect项目源程序开头声明并导入MongoDB函数库：

``` swift
import MongoDB
```

### 创建一个MongoDB数据库连接

创建到MongoDB服务器连接时，需要相应的URL，内容是IP或域名，并可选择端口号。

确定具体的连接URL之后，参考以下例子打开连接：

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
```

其中“localhost”请自行替换为实际的服务器地址。

### 定义一个数据库

一旦服务器连接成功，即可选择具体数据库：

``` swift
let db = client.getDatabase(name: "test")
```

### 定义一个MongoDB集合D

请采用以下方式定义和操作MongoDB集合：

``` swift
let collection = db.getCollection(name: "testcollection")
```

### 关闭活动的服务器连接

一旦服务器连接成功，建议采用`defer`块方式进行滞后关闭

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```
### 执行检索

请使用`find`方法在集合中检索全部有关文档：

``` swift
    let fnd = collection.find(query: BSON())

    // 初始化一个空数组用于接收格式化结果
    var arr = [String]()

    // “fnd”被定义为MongoCursor的检索记录游标，是可以遍历的
    for x in fnd! {
        arr.append(x.asString)
    }

```

有关MongoDB Collections集合类，请参考[MongoDB Collections](MongoDB-Collections.md)。
