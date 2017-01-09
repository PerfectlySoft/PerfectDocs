# 目录与路径

Perfect为服务器端的Swift语言环境提供了一个管理文件存储的便捷方法。

首先，请在源程序代码开始部分声明`PerfectLib`函数库：

``` swift
import PerfectLib
```
声明后您就随时可以使用`Dir`目录对象查询和操作文件系统。

### 设置一个目录对象参考指针

使用目录对象时，需要指定目录的绝对或相对路径：

``` swift
let thisDir = Dir("/path/to/directory/")
```

### 检查目录是否存在

使用`exists`方法检查目录是否存在。返回结果是一个布尔值，真值表示目录存在，假值表示不存在。

``` swift
let thisDir = Dir("/path/to/directory/")
thisDir.exists
```

### 返回当前目录对象的名称

调用`name`方法可以返回当前目录对象的名称。注意名称不是路径，二者并不相同！

``` swift
thisDir.name
```

### 返回到上一级目录

调用`parentDir`方法可以得到当前`Dir`目录对象所指向上一级目录。如果不存在上一级目录，则返回nil。

``` swift
let thisDir = Dir("/path/to/directory/")
let parent = thisDir.parentDir
```

### 显示当前目录对象的路径

调用`path`方法可以得到当前目录对象所在的路径。

``` swift
let thisDir = Dir("/path/to/directory/")
let path = thisDir.path
```

### 返回当前目录的UNIX权限

调用`perms`方法返回UNIX风格的目录权限信息，返回值为一个`PermissionMode`目录权限对象

``` swift
thisDir.perms
```

比如：

``` swift
print(thisDir.perms)
>> PermissionMode(rawValue: 29092)
```

### 创建一个目录

根据用户提供的权限信息创建一个目录。沿该路径下的所有目录都会根据需要一并创建。

以下操作将采用默认权限（Owner目录所有者用户具有读、写、执行权限，用户组和其它用户具有读和执行权限）创建一个新的目录。

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.create()
```

如果在创建目录时需要指定权限信息，请在调用前填写`perms`参数：

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.create(perms: [.rwxUser, .rxGroup, .rxOther])
```

如果创建目录过程中出现错误，该方法将抛出`PerfectError.FileError`错误。


### 删除一个目录

从文件系统中删除目录的方法：

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.delete()
```

如果在删除过程中出现错误，该方法会抛出`PerfectError.FileError`错误信息。

### 工作路径

### 改变当前目录对象目标指向的当前路径

请使用`setAsWorkingDir`来设置当前目录对象所指向的工作路径。

``` swift
let thisDir = Dir("/path/to/directory/")
try thisDir.setAsWorkingDir()
```

### 返回当前工作路径

返回一个新的目录对象，内容包含当前工作路径。

``` swift
let workingDir = Dir.workingDir
```

### 读取目录树结构

请以闭包为回调参数调用`forEachEntry`来遍历目录下的每一个节点，包括文件和子目录。

``` swift
try thisDir.forEachEntry(closure: {
    n in
    print(n)
})
```
