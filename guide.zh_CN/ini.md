# Perfect INI File Parser

本项目是一个简单的[INI文件](http://baike.baidu.com/item/ini文件)解析器。

本项目采用Swift 4 工具链中的SPM软件包管理器编译，是[Perfect](https://github.com/PerfectlySoft/Perfect) 项目的一部分，但也可以作为独立模块使用。

## 快速上手

配置 Package.swift 文件：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-INIParser.git", majorVersion: 3)
```

导入函数库：

``` swift
import INIParser
```

加载文件并解析为`INIParser` 对象：

``` swift
let ini = try INIParser("/path/to/somefile.ini")
```

这样就可以读取具体的变量了。

### 分节变量：

对于多数常规分节变量来说，可以使用该对象的`sections`属性进行读取，比如对于某节内容下的变量：

```
[GroupA]
myVariable = myValue
```

此时使用语句 `let v = ini.sections["[GroupA]"]?["myVariable"]` 可以得到字符串值 `"myValue"`.

### 无章节变量

但对于某些INI文件中不存在具体的分节，而是把所有变量都放在了一起，比如：

```
freeVar1 = 1
```

此时，调用匿名章节属性 `anonymousSection`即可获取变量值：

```
let v = ini.anonymousSection["freeVar1"]
```
