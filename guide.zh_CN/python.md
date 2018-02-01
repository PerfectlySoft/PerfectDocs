# Perfect - Python

本项目提供了在Swift服务器应用上直接引用Python 2.7函数库的简便方法。

本项目采用Swift Package Manager 软件包管理器编译，是[Perfect](https://github.com/PerfectlySoft/Perfect) 项目的一部分，但是也可以独立运行

在使用之前请准备好最新的Swift 4.0 工具链

## Linux 编译事项

首先请确保 libpython2.7-dev 已经在 Ubuntu 16.04 上正确安装：

```
$ sudo apt-get install libpython2.7-dev
```

## MacOS 编译事项

请确定 Xcode 9.0 以上版本已经正确安装

## 快速上手

首先在Package.swift中增加依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Python.git", majorVersion: 3)
```

然后将下列头文件导入Swift源代码：

``` swift
import PythonAPI
import PerfectPython
```

请注意在任何程序调用之前，必须调用`Py_Initialize()`函数初始化python嵌入环境：

``` swift
Py_Initialize()
```

### 导入Python函数库模块

使用 `PyObj` 类对象用于导入python模块。下列参考范例中，一个名为`/tmp/clstest.py`的脚本被动态导入到当前Swift运行环境：

``` swift
let pymod = try PyObj(path: "/tmp", import: "clstest")
```

### 访问Python变量

导入模块后，您可以使用`PyObj.load()`函数加载任何一个变量；也可以反过来用 `PyObj.save()`命令保存当前变量为一个新的值。

比如，以下python脚本中有个叫做 `stringVar` 的字符串变量：

``` python
stringVar = 'Hello, world'
```

那么要取得这个字符串的值只需要这样做：

``` swift
if let str = pymod.load("stringVar")?.value as? String {
	print(str)
	// 会打印变量的字符串值 "Hello, world!"
}
```

此时您还可以为该变量直接写入新的字符串值：

``` swift
try pymod.save("stringVar", newValue: "Hola, 🇨🇳🇨🇦！")
```


**注意** 目前，Perfect-Python仅支持如下Swift / Python数据类型自动转换：

Python 类型|Swift 类型|备注
----------|---------|-------
int|Int|
float|Double|
str|String|
list|[Any]|递归转换
dict|[String:Any]|递归转换

比如，您可以把一个字符串 `String` 转换为 `PyObj`，通过 `let pystr = "Hello".python()` 或者 `let pystr = try PyObj(value:"Hello")` 完成转换。

反过来，如果要把 `PyObj` 类转换为Swift数据类型，比如字符串，则仍然有两种方法：`let str = pystr.value as? String` 和 `let str = String(python: pystr)`。

### 执行Python函数

方法 `PyObj.call()` 用于带参数执行某个python函数。以如下python脚本为例：

``` python
def mymul(num1, num2):
	return num1 * num2
```

Perfect-Python 可以用下列方法封装并调用以上函数，您所需要注意的仅仅是其函数名称以及参数。其中函数名称用字符串代替，而参数用一个数组表达：

``` swift
if let res = pymod.call("mymul", args: [2,3])?.value as? Int {
	print(res)
	// 结果为 6
}
```

### Python类对象

请同样使用 `PyObj.load()` 函数用于家在Python类对象，但是注意后面一定要紧跟一个`PyObj.construct()` 用于初始化类对象实例。该方法同样支持用一个任意类型的数组作为参数进行对象构造。

假设如下脚本的典型python类对象 `Person`，该类有两个属性姓名`name` 和年龄`age`，还有一个名为“自我介绍”的类对象方法`intro()`:

``` python
class Person:
	def __init__(self, name, age):
		self.name = name
		self.age = age
		
	def intro(self):
		return 'Name: ' + self.name + ', Age: ' + str(self.age)
```

在Swift中初始化上述类对象的方法需要进行以下两步走：

``` swift
if let personClass = pymod.load("Person"),
    let person = personClass.construct(["rocky", 24]) {
    // person is now the object instance
  }
```

之后就可以访问类实例的属性变量和方法了，如同上文所提到的普通变量和函数调用的方法一样：

``` swift
if let name = person.load("name")?.value as? String,
    let age = person.load("age")?.value as? Int,
    let intro = person.call("intro", args: [])?.value as? String {
      print(name, age, intro)
}
```

### 回调函数

参考以下python代码，此时如果执行 `x = caller('Hello', callback)` 则可以将函数作为参数进行回调:

``` python
def callback(msg):
    return 'callback: ' + msg

def caller(info, func):
    return func(info)
```

在Swift中等效的代码平淡无奇，只不过将待调函数作为参数而已：:

``` swift
if let fun = pymod.load("callback"),
   let result = pymod.call("caller", args: ["Hello", fun]),
   let v = result.value as? String {
   		print(v)
   	// 结果是 "callback: Hello"
}
```