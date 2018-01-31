# MongoDBStORM

## 相关例子

* [MongoDBStORM-Demo](https://github.com/PerfectExamples/MongoDBStORM-Demo)
* [Perfect-Session-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Session-MongoDB-Demo)
* [Perfect-Turnstile-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MongoDB-Demo)


## 使用方法

首先是在您的项目Package.swift中增加如下依存关系，注意该函数库包括了所有必要的其他组件，比如数据库连接件。

``` swift
.Package(url: "https://github.com/SwiftORM/MongoDB-Storm.git", majorVersion: 3)
```

## 连接到数据库服务器

连接到服务器需要设置信息参考范例如下：

``` swift
MongoDBConnection.host = "localhost"
MongoDBConnection.port = 27017
MongoDBConnection.ssl = true
MongoDBConnection.database = "mydb"

// 用户名密码
// 只有在选项 authModeType = .standard 时才用得上
MongoDBConnection.username = "username"
MongoDBConnection.password = "secret"
MongoDBConnection.authdb = "authenticationSource"

// 身份认证模式：
// .none (默认), 或 .standard
MongoDBConnection.authModeType = .none
```

输入完上述信息后，将在第一次使用对象类时自动激活连接。

``` swift
let obj = User()
```

## MongoDBStORM 支持的方法

### 数据库连接

`MongoDBConnection` - 设置芒果数据库的连接参数，包括主机名、数据库信息、端口、加密证书SSL等等（默认不使用SSL，端口号是27017）。

### 创建集合

创建集合的请参考以下初始化函数，类对象（集合）名称可以自行决定：

``` swift
	override init() {
		super.init()
		_collection = "users_demo"
	}
```

注意与其他数据库版本的StORM关系管理自动化方法不同，芒果数据库不需要使用`setup()`函数用于创建集合。MongoDB会根据需要在数据插入时自动创建。

> **注意** 在类对象中 **主索引** 是第一条属性。

### 保存数据

`save()` - 保存对象的当前数据值。如果ID编号指定，则采用覆盖更新的方法保存；否则将创建新的数据记录。

### 获取数据

`get()` - 获取记录，此时假定ID编号已经提前设置好。

`get(String)` - 根据具体编号获取记录／

`find([String: Any])` - 执行查询。比如、 `try find(["username":"joe"])` 会检索所有用户名为"joe"的记录。
		
此外，`find` 还包括：

*  `cursor: StORMCursor` - 可选的游标`cursor`对象，参考 [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Cursor.md)

### 删除数据

`delete()` - 删除当前数据。假设该对象id编号提前准备好。


