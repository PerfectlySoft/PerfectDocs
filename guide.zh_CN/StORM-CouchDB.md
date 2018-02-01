# CouchDBStORM 数据库关系自动管理

### 在项目中引用

请在您当前工程的 Package.swift 中增加以下依存关系：

``` swift
.Package(url: "https://github.com/SwiftORM/CouchDB-Storm.git", majorVersion: 3)
```


## 连接到数据库

请设置如下属性以连接数据库：

``` swift
CouchDBConnection.host = "localhost"
CouchDBConnection.username = "username"
CouchDBConnection.password = "secret"
CouchDBConnection.port = 5984
CouchDBConnection.ssl = true
```

一旦设置信息确认，则数据连接会在对象应用过程中自动创建。

``` swift
let obj = User()
```

## CouchDBStORM 支持的函数方法

### 数据连接

`CouchDBConnection` - 设置连接参数，用于访问 CouchDB 服务器。请确认连接所需的主机名、用户名和密码。如果采用非标准端口和加密传输SSL时，才需要设置端口和SSL。默认情况下SSL为false，即不使用SSL，默认端口为 5984。

### 创建数据库

对象所用的工作数据库是自动嵌入的，可以根据需要自行命名：

``` swift
	override open func database() -> String {
		return "我的数据库名字"
	}
```

`setup()` - 创建数据库

> **⚠️注意⚠️** 其中，**主索引键** 是类对象内定义的第一个属性。

#### 自定义表创建SQL语句

`setup(String)` - 用于手工创建数据表，将会覆盖任何其他 StORM 设置语句。

### 保存数据

`save(rev: String = "")` - 保存对象数据。如果定义了一个ID编号，则自动执行更新，否则将插入一个新的文档。rev参数用于说明文档修订版本号。如果为该参数为空的话，则会自动使用`_rev`属性值代替修订版本号。

`save(rev: String = "") {id in ... }` - 保存对象数据。如果定义了一个ID编号，则自动执行更新，否则将插入一个新的文档。rev参数用于说明文档修订版本号。如果为该参数为空的话，则会自动使用`_rev`属性值代替修订版本号。闭包用于在新创建文档时返回新文档编号。

`create()` - 显式要求创建新的数据库对象（而不是更新）。修订版本号也会在保存完毕后自动设置。

### 读取数据

`get()` - 在假设id已经设置完成的条件下获取文档。

`get(String)` - 根据id获取文档。

`find([String: Any])` - 根据查询条件获取数据。比如，`try find(["用户名":"老张"])` 就能够过滤所有文档找到用户名等于“老张”的那个文档。关于查询选择条件，详见 [http://docs.couchdb.org/en/2.0.0/api/database/find.html#find-selectors](http://docs.couchdb.org/en/2.0.0/api/database/find.html#find-selectors)
		
此外，`find`还能包括：

*  `cursor: StORMCursor` - 可选的 `cursor` 游标对象是一个 [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Cursor.md)

### 删除对象

`delete()` - 删除当前对象，假设 id 编号和 _rev 修订号已经设置。


