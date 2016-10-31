# ZIP压缩

Perfect为开发者提供了一个以C语言库实现的迷你zip压缩工具箱，内容是一组非常方便的函数接口。使用时，请导入PerfectZip。

## 准备工作

对于MacOS用户，请使用homebrew安装minizip:

```
brew install minizip
```

在 Ubuntu下, 使用apt-get命令安装minizip:

```
apt-get install minizip
```

除PerfectLib 之外，还需要在Package.swift文件中说明当前项目对Perfect-Zip 的依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Zip.git", versions: Version(0,0,0)..<Version(10,0,0))
```

## 使用 Perfect Zip

Perfect Zip函数库中只有两个核心函数：压缩文件和解压缩文件

### 声明Zip类实例

在开始压缩或者解压缩之前，首先需要创建zip类实例：

``` swift
let myVar = Zip()
```


### Zip 压缩

如果需要压缩一个文件，请使用`.zipFiles(...)`方法：/p>

``` swift
zipFiles(
    paths: [String],
    zipFilePath: String,
    overwrite: Bool,
    password: String?
) -> ZipStatus
```

该方法调用后会返回一个`ZipStatus`枚举变量，用于说明压缩操作是成功还是失败。

#### 参数说明：

* **paths:** 用于保存待压缩文件所在路径信息的数组
* **zipFilePath:** 压缩后文件的保存路径
* **overwrite:** 如果压缩后的文件已经存在，本次压缩是否将已存在文件。此处如果设置为真，则覆盖；否则忽略。
* **password:** 可选参数；如果需要对该压缩文件设置密码保护，则该字符串用于存储密码。


### UnZip 解压缩

如果要解压缩一个文件，请使用`.unzipFile(...)`方法

``` swift
unzipFile(
    source: String,
    destination: String,
    overwrite: Bool,
    password: String = ""
) -> ZipStatus
```

该方法调用后会返回一个`ZipStatus`枚举变量，用于说明解压缩操作是成功还是失败。

#### 参数说明

* **source:** 压缩文件的路径信息
* **destination:** 解压缩文件的目标路径
* **overwrite:** 如果目标路径已存在，是否需要覆盖。如果此处变量设置为真则覆盖；否则忽略
* **password:** 可选参数；用于解压缩的密码。


### ZipStatus 操作结果状态

压缩/解压操作返回结果枚举值 `ZipStatus` 参考如下：

* .FileNotFound
* .UnzipFail
* .ZipFail
* .ZipCannotOverwrite
* .ZipSuccess

该枚举类型还包括一个 `.description` 变量，用于返回每个返回值所代表的具体含义。

* .FileNotFound - **"文件不存在"**
* .UnzipFail - **"解压缩失败"**
* .ZipFail - **"压缩失败"**
* .ZipCannotOverwrite - **"无法覆盖目标文件"**
* .ZipSuccess - **"操作成功"**



## 使用方法

下面的源代码演示了如何压缩一个文件

``` swift
import PerfectZip

let myZip = Zip()

let thisZipFile = "/path/to/ZipFile.zip"
let sourceDir = "/path/to/files/"

let ZipResult = myZip.zipFiles(
    paths: [sourceDir],
    zipFilePath: thisZipFile,
    overwrite: true, password: ""
)
print("压缩结果：\(ZipResult.description)")
```

下面的源代码演示了如何解压缩一个文件

``` swift
import PerfectZip

let myZip = Zip()

let sourceDir = "/path/to/files/"
let thisZipFile = "/path/to/ZipFile.zip"

let UnZipResult = myZip.unzipFile(
    source: thisZipFile,
    destination: sourceDir,
    overwrite: true
)
print("解压缩结果：\(UnZipResult.description)")
```
