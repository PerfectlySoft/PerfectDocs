# 文件操作
Perfect为服务器端的Swift语言环境提供了文件访问操作的便捷接口函数。

首先，请确保`PerfectLib`已经在您的Swift源程序开始部分完成声明：

``` swift
import Perfectib
```
声明完成之后，即可开始使用`File`文件对象来实现在文件系统中的查询和文件操作。

### 配置File文件对象

使用File文件对象时，请指定文件的绝对或相对路径：

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
```


### 打开文件用于读写操作

> **注意：** 在写入文件操作之前（即使是个新文件）文件需要对应的权限才能打开。

打开一个文件：

``` swift
try thisFile.open(<OpenMode>,permissions:<PermissionMode>)
```

写入文件的例子：

``` swift
let thisFile = File("helloWorld.txt")
try thisFile.open(.readWrite)
try thisFile.write(string: "你好！")
thisFile.close()
```

对于文件打开的具体模式[OpenMode](#OpenMode) 和文件权限模式 [PermissionMode](PermissionMode) ，请参考本文附录的详细定义。

### 检查文件是否存在

请使用`exists`方法检查文件是否存在。该方法返回一个布尔值，真表示存在，假表示不存在。

``` swift
thisFile.exists
```


### 获取文件修改时间

调用下面的函数将返回一个整数，其含义为自格林威治时间1970/01/01 00:00:00到最后一次修改文件的时间之间的秒数：

``` swift
thisFile.modificationTime
```

### 文件路径

无论文件参考对象是如何定义的，文件所在的绝对路径和内部参考路径都可以通过以下访问获取具体信息：

返回系统内部参考路径"internal reference"：

``` swift
thisFile.path
```

返回文件的绝对路径（物理路径）方法（如果当前文件为符号链接，则同样其链接也会被解析为绝对路径）：

``` swift
thisFile.realPath
```

### 关闭一个文件

一旦文件被打开并执行读写操作，随时可以在程序内选择用close方法关闭，或使用`defer`方法挂起：

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
// 在此处进行文件读写操作处理
thisFile.close()
```


### 删除一个文件

如果需要从文件系统中删除一个文件，请使用`delete()`方法。

``` swift
thisFile.delete()
```

调用删除方法时会自动关闭文件，因此不需要提前进行额外的`close()`关闭操作。



### 查询文件大小尺寸

调用`size`方法可以返回文件大小的字节数，返回值为整数。

``` swift
thisFile.size
```

### 判断文件是否为一个符号链接

用下面的方法来判断当前文件对象是否为一个符号链接，如果是符号链接则返回值为布尔类型的`true`,真值，反之为`false`假。

``` swift
thisFile.isLink
```

### 判断当前文件对象是否为一个目录

用`isDir` 方法来判断当前文件对象是否为一个目录节点，如果是则返回值为布尔类型的`true`,真值，反之为`false`假。

``` swift
thisFile.isDir
```

### 获取文件的UNIX权限信息

调用`perms` 函数可返回文件的权限信息，返回值为一个`PermissionMode`对象。

``` swift
thisFile.perms
```

比如：

``` swift
print(thisFile.perms)
>> PermissionMode(rawValue: 29092)
```

## 读取文件内容

### readSomeBytes

根据指定的字节数量读取文件内容：

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: <Int>)
```

#### 参数说明

该方法的参数为计划读取的字节数量。比如，下面的例子说明了如何从文件中读取10字节数据：


``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: 10)
print(contents)
>> [35, 32, 80, 101, 114, 102, 101, 99, 116, 84]
```



### readString

`readString`方法能够将整个文件的数据读取到一个字符串：

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readString()
```

## 文件写操作、拷贝和移动

> **注意** 在写文件之前（即使是一个新文件）必须要确认文件操作具备响应的权限。


### 向文件内写入字符串

使用`write`方法以UTF-8编码格式向文件创建或写入一个字符串。如果写入成功，则返回具体写入的字节数。

注意该方法会使用`@discardableResult`属性，所以如果需要的话，即便没有对该属性赋值，也会在调用过程中被使用。

``` swift
let bytesWritten = try thisFile.write(string: <String>)
```

### 将字节数组写入到一个文件

字节数组`bytes`同样可以直接写入到一个文件。该方法如果调用成功则返回实际写入的字节数。

注意该方法会使用`@discardableResult`属性，所以如果需要的话，即便没有对该属性赋值，也会在调用过程中被使用。

``` swift
let bytesWritten = try thisFile.write(
    bytes: <[UInt8]>,
    dataPosition: <Int>,
    length: <Int>
    )
```

#### 参数说明

* **bytes:** 待写入的无符号8位整型数组
* **dataPosition:** *可选参数* 在`bytes`对象中待写入字节的偏移量。如果该参数不为0，则写入时从字节数组的该参数指定字节处开始写入，之前内容会被忽略
* **length:** *可选参数* 需要写入的具体字节数量


### 移动一个文件

文件对象一旦定义成功，就可以随时调用`moveto`方法，将文件移动到一个新的路径位置上。该方法也可以用于给文件改名。操作完成后，该方法返回一个代表新位置的新的文件对象。

``` swift
let newFile = thisFile.moveTo(path: <String>, overWrite: <Bool>)
```

#### 参数说明

* **path:** 移动的目标路径
* **overWrite:** *可选参数* 如果目标路径文件已经存在，则进行覆盖操作。默认值为假`false`

## 错误处理

该方法会在出错的情况下抛出`PerfectError.FileError`异常

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.moveTo(path: "/path/to/file/goodbyeWorld.txt")
```

### 拷贝文件

类似于`moveTo`方法，`copyTo`可以将文件拷贝到新的目录，而且通过可选参数决定如果目标目录已存在同名文件的情况下是否覆盖。不同的是，复制文件不会删除原文件。操作完成后，将返回一个新的文件对象，代表其复制品的路径信息。

注意该方法会使用`@discardableResult`属性，所以如果需要的话，即便没有对该属性赋值，也会在调用过程中被使用。

``` swift
let newFile = thisFile.copyTo(path: <String>, overWrite: <Bool>)
```
#### 参数说明

* **path:** 待拷贝的目标路径
* **overWrite:** *可选参数* 如果设置为真则表示在目标路径已存在同名文件的情况下，用新文件进行覆盖。默认情况下该参数值为`false`

#### 错误处理

该方法会在出错的情况下抛出`PerfectError.FileError`异常

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.copyTo(path: "/path/to/file/goodbyeWorld.txt")
```

### 文件锁函数

文件锁函数允许文件的部分内容*sections*采用建议锁（部分文献称为“劝告锁”，即只有协同操作进程才能实现的锁定机制，其它非数据库函数的非协同操作进程将忽略这类锁的存在而进行强行读写——译者注）*advisory-mode locks*。

一旦文件关闭或进程推出，所有关于该文件的锁全部自动解除。

> **注意：** 这一类锁并非系统锁，而且不会阻止其它试图对目标文件进行写操作的进程，因为这类所为建议锁“advisory-mode locks”。

### 锁定一个文件

尝试对文件对象的当前位置开始的数个字节内容进行锁定。该函数将阻塞当前线程直至加锁操作完成。

``` swift
let result = try thisFile.lock(byteCount: <Int>)
```

### 文件解锁

尝试对文件对象当前位置开始的数个字节内容进行解锁。

``` swift
let result = try thisFile.unlock(byteCount: <Int>)
```

### 尝试锁定文件

尝试对文件对象的当前位置开始的数个字节内容进行锁定。该函数如果在文件已经加锁的情况下会抛出一个异常，单不会阻塞当前线程。

``` swift
let result = try thisFile.tryLock(byteCount: <Int>)
```

### 测试文件锁

测试目标的字节区段是否已经被锁定，返回值为布尔类型。如果已经锁定则返回真，否则返回假。

``` swift
let isLocked = try thisFile.testLock(byteCount: <Int>)
```

## OpenMode

文件的打开模式OpenMode由以下枚举定义：

* .read：以只读方式打开文件
* .write：以只写方式打开文件；如果文件不存在则创建一个新文件
* .readWrite：以读写方式打开文件；如果文件不存在则创建一个新文件
* .append：以读写方式打开文件；如果文件不存在则创建一个新文件；将读写起始标志设置到文件尾部，即文件内容追加
* .truncate：以读写方式打开文件；如果文件不存在则创建一个新文件；同时将文件尺寸设置为0

下面的例子为文件写操作：

``` swift
let thisFile = File("helloWorld.txt")
try thisFile.open(.readWrite)
try thisFile.write(string: "你好！")
thisFile.close()
```

## PermissionMode

文件的权限模式PermissionMode是由单个选项或者由多个选项组成的数组来表达。

比如，以下列指定权限创建一个目录：

``` swift
let thisDir = Dir("/path/to/dir/")
do {
    try thisDir.create(perms: [.rwxUser, .rxGroup, .rxOther])
} catch {
    print("创建失败")
}
//或者
do {
    try thisDir.create(perms: .rwxUser)
} catch {
    print("发现异常")
}
```

### PermissionMode 权限模式选项

* `.readUser`：所有者只读
* `.writeUser`：所有者可写
* `.executeUser`：所有者可执行
* `.readGroup`：所有者所在组可读
* `.writeGroup`：所有者所在组可写
* `.executeGroup`：所有者所在组可执行
* `.readOther`：其它所有用户均可读
* `.writeOther`：其它所有用户均可写
* `.executeOther`：其它所有用户均可执行
* `.rwxUser`：所有者用户可以读、写、执行
* `.rwUserGroup`：所有者用户所在组可以读、写
* `.rxGroup`：所有者用户所在组可以读、执行
* `.rxOther`：其它所有用户可以读、执行
