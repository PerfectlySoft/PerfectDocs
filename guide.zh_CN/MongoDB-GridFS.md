# MongoDB GridFS 文件网格系统

GridFS 文件网格系统是用于超出 16MB 以上的大型 BSON 文档而设计的文件存取系统。Perfect-GridFS 函数库提供两种不同的对象类用于访问 MongoDB 的 GridFS 文件系统：

- GridFS：提供文件列表、检索、删除、上载、下载和删除整个文件系统的功能
- GridFile：提供文件信息、下载、局部读写和设置文件读写位置的功能

## macOS 编译注意事项
请⚠️注意⚠️本模块需要 MongoDB 的 C语言模块在1.7.1版本之后的特殊修订，详见[BSON-API 无法设置文件属性问题的修订](https://github.com/mongodb/mongo-c-driver/pull/415)

## GridFS 类

GridFS 程序基于 MongoClient 类之上。首先，请确定已经启动了一个 Monge Client 对象的实例，然后调用该实例的 `gridFS()` 方法，并在调用时注意提供必要的数据库名，以及可选的前缀参数：

``` swift
// 假设 let client = try MongoClient(uri: "mongodb://...")
let gridfs = try client.gridfs(database: "test")
```
默认情况下可以忽略 prefix 前缀参数，但是也可以根据需要自行设置：

``` swift
let gridfs = try client.gridfs(database: "test", prefix: "clip")
```


### GridFS 类方法

GridFS 对象类提供以下方法

- list(filter 可选参数): 列出服务器上所有文件 ／ 部分文件（如果包含了可选的过滤器参数）
- drop(): 删除数据库上的文件系统以及所哟文件
- search(filename): 在服务器上搜索一个名为 filename 的文件
- delete(filename): 从服务器上删除一个名为 filename 的文件
- upload(from, to): 把本地文件 `from` 上传到服务器上并命名为 `to` 
- download(from, to): 把服务器文件 `from` 下载到本地路径 `to`
- close(): 关闭文件系统

#### 文件清单

`list()` 方法能够返回一个 `[GridFile]` 数组。如果希望缩小清单范围，可以使用一个类型为 BSON 的 filter 参数：

``` swift
func list(filter: BSON? = nil) throws -> [GridFile]
```

默认情况下，`list()` 并不要求输入过滤器参数，但是实际上内部程序自动生成了一个默认的 bson 结构，并将查询结果设置为“按照字母顺序排序”：

``` swift
let list = try gridfs.list()
print("\(list.count) file(s) found")
```

#### 删除文件系统

使用 `drop()` 方法能够要求 GridFS 删除当前数据库上的整个文件系统以及所有相关文件。

``` swift
gridfs.drop()
```


#### 检索文件

请使用 `search()` 方法检索文件并获取文件信息、或在检索结果基础之上执行更加复杂的大文件操作（见 `GridFile` 对象类）。如果文件没找到，则会返回一个 `MongoClientError` 异常：

``` swift
let f = try gridfs.search(name: "file.name.on.server")
// 打印文件对象内部序列号
print(f.id)
// 输出文件长度，以字节计算
print(f.length)
```

#### 删除文件

调用 `delete()` 方法能够根据文件名称删除文件。如果文件不存在，则会返回`MongoClientError` 异常：


``` swift
try gridfs.delete(name: "file.name.on.server")
```

#### 上载文件

输入本地路径名称和预期服务器名称则可以上传一个文件，如果失败会报错：

``` swift
let gridfile = try gridfs.upload(from: "/path/to/local.file", to: "name.on.server.type", contentType: "primaryType/subType", md5: "MD5 string", metaData: BSON(json: "meta data in a json string"), aliases: BSON(json: "aliases in a json string")
print(gridfile.length)
```

参数：
- from: 待上传的本地文件路径
- to: 目标存放在服务器上的文件名
- contentType: 文件类型，可选。默认值为 `text/plain`
- md5: 可选字符串，代表文件的MD5校验值。⚠️请注意⚠️可以使用[Perfect 	COpenSSL](https://github.com/PerfectlySoft/Perfect-COpenSSL) 和 [Perfect 	COpenSSL Linux Edition](https://github.com/PerfectlySoft/Perfect-COpenSSL-Linux) 计算字符串的MD5
- metaData: 可选的BSON()格式，代表文件的元数据属性
- aliases: 可选的BSON()格式，代表文件的别名


#### 下载文件

只需确定目标服务器文件名称以及本地存储路径即可实现文件下载。如果文件不存在或者网络出问题，则会抛出`MongoClientError`异常：

``` swift
do {
	let bytes = try gridfs.download(from: "name.on.server", to: "/local/path/downloaded.name")
	print("总共下载了 \(bytes) 个字节")
}catch(let err) {
	print("下载中出现错误： \(err)")
}
```

#### 关闭文件系统连接

由于 Swift 的高效垃圾回收机制，实用中并不要求手工关闭文件系统连接。但是用户当然可以在使用完成后强制关闭文件系统：

``` swift
defer {
	gridfs.close()
}//end defer
```

## GridFile 对象类

GridFS 的方法 `search()` 和 `list()` 会分别返回 `GridFile`类对象 和 `[GridFile]` 数组。GridFile 用于管理文件信息以及处理更复杂的大文件操作，如文件位置游标及局部内容分片读写等等。

### GridFile 属性（只读）：

- id: oid 标识字符串。
- md5: md5 校验字符串。
- aliases: BSON 类型的属性，代表文件别名。
- contentType: 代表文件内容类型的字符串。
- length: 64位整型的文件长度。
- uploadDate: 64位整型时间戳，Unix纪元格式。
- fileName: 存储在服务器上的文件名。
- metaData: BSON 类型的元数据。

### GridFile 方法：

除了上述属性之外，`GridFile` 还包括以下方法：

- download(): 直接下载整个文件
- tell(): 获取当前文件游标指向位置，UInt64类型整数
- seek(cursor, 可选 whence): 根据可选参数 whence 确定文件游标新位置
- paritallyRead(amount, 可选 timeout): 在规定的可选时间内，从当前游标位置开始读取文件部分内容并返回到一个二进制缓冲区[UInt8]
- partiallyWrite(bytes, optional timeout): 在规定的可选时间内，在当前游标位置写入一个二进制缓冲区
- close(): 关闭文件连接

#### download() 下载文件

GridFile对象的下载方法 `download()` 是文件系统 `GridFS.download()`方法的基础，后者在内部调用前者。如果已经获取了GridFile 对象句柄，那么可以直接设置本地路径来下载该文件：

``` swift
do {
	let bytes = try gridfile.download(to: "/local/path/downloaded.name") 
	print("共下载了 \(bytes) 字节")
}catch(let err) {
	print("下载时出现异常： \(err)")
}
```

#### tell() 获取游标

`tell()` 方法用于取得当前文件实例的缓冲区游标，返回类型为64位无符号整数：

``` swift
let position = gridfile.tell()
print(“当前文件指向 \(position)”)
```

#### seek() 设置游标

`seek()` 方法用于设置 `GridFile`对象的游标位置，可选参数`whence`用于说明相对位置所用的参考点。`.begin`为默认参考点，代表从文件头开始计算相对位置；而`.current`表示偏移量从目前游标所在位置开始计算；`.end`则代表从文件尾部开始计算偏移量：

``` swift
enum Whence {
    // 从文件头开始计算偏移量
    case begin
    // 从当前游标所在文件位置计算相对偏移量
    case current
    // 从文件尾部最后一个字节开始反向计算相对偏移量
    case end
 }//end whence
  
func seek(cursor: Int64, whence:Whence = .begin) throws
```

比如，下面的代码将文件游标指向了从文件头开始1兆字节所在位置：

``` swift
try gridfile.seek(1048576)
```

#### partiallyRead() 部分读取

为了能够在线处理大型文件，GridFile 提供了文件局部读写功能。首先是部分读取函数：

``` swift
func partiallyRead(amount: UInt32, timeout:UInt32 = 0) throws -> [UInt8] 
```

参数 `amount` 是计划读取的字节数量，而参数 `timeout` 表示等待执行该操作所需要的时间毫秒数。如果读取失败，则会抛出 `MongoClientError` 异常。请注意 ⚠️*NOTE*⚠️ 任何关于文件的操作，如游标设置函数 `seek()`、部分读取和部分写入函数`paritallyRead()`、 `partiallyWrite()`都会改变文件游标位置。请参考下列程序理解文件游标的概念：

``` swift
let file = try gridfs.search("a.large.file")
let cursor = file?.tell()
print("在没有任何操作以前，游标取值应该是零。当前游标值 = \(cursor)")
let oneMegaByte = 1048576
// 将游标指向距离文件起始位置100兆字节处
try file?.seek(cursor: Int64(oneMegaByte * 100))
// 然后从这个位置读取1兆数据
let bytes = try file?.partiallyRead(amount: UInt32(oneMegaByte))
```

#### partiallyWrite() 部分写入

方法 `partiallyWrite()` 部分写入是与上面的 `partiallyRead()`相反的操作。如果写入失败，则会抛出一个 `MongoClientError` 异常：

``` swift
public func partiallyWrite(bytes:[UInt8], timeout:UInt32 = 0) throws -> Int
```

第一个参数`bytes`是待写入文件的缓冲区，而可选参数`timeout`则是等待写入操作的毫秒数时间，如果忽略超时参数则函数在调用后立刻返回。 ⚠️*再次注意*⚠️，任何关于文件的操作，如游标设置函数 `seek()`、部分读取和部分写入函数`paritallyRead()`、 `partiallyWrite()`都会改变文件游标位置：

``` swift
// 将文件游标指向距离文件尾部1K字节处：
try file?.seek(cursor: 1024, whence: .end)
let buffer:[UInt8] = [1,2,3,4,5]
// 写入一组数据
let totalWritten = try file?.partially(bytes: buffer)
print(totalWritten)
```
如果成功的话，上述代码会在距离文件尾部1K位置写入5个字节。

#### close() 关闭文件

由于 Swift 的高效垃圾回收机制，实用中并不要求手工关闭文件连接。但是用户当然可以在使用完成后强制关闭文件：

``` swift
defer {
	gridfile.close()
}//end defer
```

## 关于性能

大型文件读写通常都是非常耗时的，因此，请尝试使用线程管理`Threading.dispatch {}`，避免上传下载文件时阻塞主程序，或者尽量使用小片文件的读写`partiallyRead()/Write()`。关于线程的更多信息，请查看[Perfect Threading](thread.md)