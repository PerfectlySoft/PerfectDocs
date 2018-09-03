# MySQL

MySQL连接库提供了对MySQL的使用封装，允许您的Perfect应用程序于MySQL数据库进行交互操作。

### 系统要求

### macOS

需要使用Homebrew安装MySQL。

```
brew install mysql@5.7
```

如果需要安装Homebrew，请用下面的命令行进行安装：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

⚠️**注意**⚠️ Perfect目前支持的最后一个mysql自制软件版本是5.7，所以当你的构建中发现一些遗漏类型问题时，请尝试使用此命令：

```
$ brew install mysql@5.7 && brew link mysql@5.7 --force
```



### Linux

请确认您的Linux系统已经安装了MySQL 5.6版本以上的libmysqlclient-dev库文件：

```
sudo apt-get install libmysqlclient-dev
```

请注意Ubuntu 14默认安装的版本无法编译这个套件，需要手工安装MySQL客户端5.6或以上版本。

### 设置

请在您的Package.swift文件中增加“Perfect-MySQL”用于说明调用库函数的依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-MySQL.git", majorVersion: 3)
```

### 声明和导入

为了使用MySQL函数库，首先需要在您开发的源程序开始部分增加声明和导入操作：

``` swift
import PerfectMySQL
```

### 快速上手

### 访问数据库

为了访问数据库，请将您的数据库配置为以下的用户名和密码：

``` swift
let testHost = "127.0.0.1"
let testUser = "test"
let testPassword = "password"
let testDB = "schema"

// 显然用户名和密码需要根据您自己安装的数据库来决定
```

有两种通用的方法可以用于连接MySQL。第一种是，首先跳过选择具体的数据库Schema（指的是MySQL内用户可以创建多个Schema，具体的数据表格是保存在Schema中——译者注），这么做的好处是可以在之后的程序内选择具体的数据库Schema，特别适用于您的程序如果需要操作多个Schema：

``` swift
func fetchData() {
	let dataMysql = MySQL() // 创建一个MySQL连接实例
	let connected = mysql.connect(host: testHost, user: testUser, password: testPassword)
	guard connected else {
		// 验证一下连接是否成功
		print(mysql.errorMessage())
		return
	}

	defer {
		mysql.close() //这个延后操作能够保证在程序结束时无论什么结果都会自动关闭数据库连接
	}

	// 选择具体的数据Schema
   guard mysql.selectDatabase(named: testDB) else {
   		Log.info(message: "数据库选择失败。错误代码：\(mysql.errorCode()) 错误解释：\(dataMysql.errorMessage())")
		return
	}
}
```

另外一种方式是在连接数据库时将具体的数据库Schema选择作为参数直接传入创建连接的调用过程，不必单独在数据连接后才选择Schema：

``` swift
func fetchData() {
	let mysql = MySQL() // 创建一个MySQL连接实例
	let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)

	guard connected else {
   		// 验证一下连接是否成功
		print(mysql.errorMessage())
		return
	}

	defer {
		mysql.close() //这个延后操作能够保证在程序结束时无论什么结果都会自动关闭数据库连接
	}
}
```

### 创建数据表格

Perfect允许在程序内创建表格。进一步继续前面的例子，很容易实现创建表格的操作

``` swift
func setupMySQLDB() {
	let mysql = MySQL() // 创建一个MySQL连接实例
	let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)

	guard connected else {
		// 验证一下连接是否成功
		print(mysql.errorMessage())
		return
	}

	defer {
		mysql.close() //这个延后操作能够保证在程序结束时无论什么结果都会自动关闭数据库连接
	}

	// 执行查询或者创建表格
  	let sql = """
  	CREATE TABLE IF NOT EXISTS ticket (
	id VARCHAR(64) PRIMARY KEY NOT NULL,
	expiration INTEGER)
	"""
	guard mysql.query(statement: sql) else {
        // 验证是否创建成功
        print(mysql.errorMessage())
        return
   }
}
```

### 运行数据库查询

从数据库中查询是最基本的操作，也相对简单。查询完成之后就可以保存结果记录集并根据结果执行进一步的程序。以下例子中我们假定当前数据库Schema存在一个名为options（即选项的意思）的数据表，该表有一个id字段，一个name（text类型）字段和value字段（text类型）：

``` swift
func fetchData() {
	let mysql = MySQL() // 创建一个MySQL连接实例
	let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)
	
	guard connected else {
		// 验证一下连接是否成功
		print(mysql.errorMessage())
		return
	}

	defer {
		mysql.close() //这个延后操作能够保证在程序结束时无论什么结果都会自动关闭数据库连接
	}

	// 运行查询（比如返回在options数据表中的所有数据行）
	let querySuccess = mysql.query(statement: "SELECT option_name, option_value FROM options")

	// 确保查询完成
	guard querySuccess else {
		return
	}

	// 在当前会话过程中保存查询结果
	let results = mysql.storeResults()! //因为上一步已经验证查询是成功的，因此这里我们认为结果记录集可以强制转换为期望的数据结果。当然您如果需要也可以用if-let来调整这一段代码。

	var ary = [[String:Any]]() //创建一个字典数组用于存储结果

	results.forEachRow { row in
		let optionName = getRowString(forRow: row[0]) //保存选项表的Name名称字段，应该是所在行的第一列，所以是row[0].
		let optionValue = getRowString(forRow: row[1]) //保存选项表Value字段
		ary.append("\(optionName)":optionValue]) //保存到字典内
	}
}
```

## MySQL 服务器 API 函数

MySQL服务器API函数提供了连接到服务器实例并展开相关工作的一组工具，包括基本连接、关闭连接、查询数据库和数据表，运行查询（实际上是对整个[SQL查询语句API](#mysql-statements-api)的封装。但不同是的返回的结果记录集是通过对[结果记录集API](#mysql-results-api)的封装完成。查询语句当然也可以通过[SQL查询语句API](#mysql-statements-api)完成，为您使用MySQL类提供更详细更简便的查询方法。

### init 类构造函数

``` swift
public init()
```

创建一个MySQL类实例并允许程序访问MySQL数据库。

### ⚠️文字编码注意事项⚠️

如果您的数据库中包含非ascii码，比如中文，那么必须要在连接前设置下列选项：

``` swift
setOption(.MYSQL_SET_CHARSET_NAME, "utf8")
```

### close

``` swift
public func close()
```

关闭当前MySQL连接。通常可以使用验证连接后进行延缓关闭（defer），这样做能够保证无论在任何情况下都能关闭当前数据库会话。

### clientInfo 客户端信息查询

``` swift
public static func clientInfo() -> String
```

调用上面的函数可以获取一个用于代表目前MySQL客户端版本的字符串，比如”5.7.x“，或者根据您当前系统所安装的版本返回具体信息。

### errorCode & errorMessage 错误代码和错误信息

``` swift
public func errorCode() -> UInt32
```

``` swift
public func errorMessage() -> String
```

错误代码和错误信息对于调试程序来说非常有用。详见[服务器错误信息对照表（英文版）](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html)。错误代码和错误信息的获取与解释对于连接数据库运行查询来说非常重要，举例如下：

``` swift
let mysql = MySQL()
let connected = mysql.connect(host: dbHost, user: dbUser, password: dbPassword, db: dbName)
guard connected else {
	// 验证是否成功，如果不成功则打印错误信息
	print(mysql.errorMessage())
	return
}
```

在上面的例子里面，如果数据库连接失败则在控制台上会输出错误信息。

### serverVersion 服务器版本

``` swift
public func serverVersion() -> Int
```

调用后会返回一个代表当前MySQL服务器版本号的整数。

### connect 连接数据库

``` swift
public func connect(host hst: String? = nil, user: String? = nil, password: String? = nil, db: String? = nil, port: UInt32 = 0, socket: String? = nil, flag: UInt = 0) -> Bool
```

连接到MySQL数据库。调用时至少要提供的基本验证信息（通常是主机名、用户和密码）。作为选项，您还可以指定连接端口、具体的数据库Schema或连接套接字。在连接中以参数确定数据库Schema并不是强制要求的，您可以在完成连接后使用[selectDatabase()](#selectDatabase)方法选择具体的数据库Schema。

### selectDatabase

``` swift
public func selectDatabase(named namd: String) -> Bool
```

即从当前活动的MySQL连接中选择目标数据库Schema。

### listTables列出所有数据表

``` swift
public func listTables(wildcard wild: String? = nil) -> [String]
```

调用后，将返回一个字符串数组，数组中的每一个元素代表了目标数据库Schema中的不同数据表名称。

### listDatabases 列出所有数据库

``` swift
public func listDatabases(wildcard wild: String? = nil) -> [String]
```

调用后，将返回一个字符串数组，数组中的每一个元素代表了一个当前MySQL服务器上可以访问的数据库Schema。

### commit提交事务

``` swift
public func commit() -> Bool
```

在事务关联的一组查询准备好之后，提交事务（完成交易）

### rollback事务回滚

``` swift
public func rollback() -> Bool
```

取消交易，回滚数据。

### moreResults查看更多结果记录集

``` swift
public func moreResults() -> Bool
```

调用`mysql_more_results`检查一个查询操作是否包含了更多的结果记录集。

### nextResult 查看下一个结果记录集

``` swift
public func nextResult() -> Int
```

在多结果查询中返回下一个结果记录集。通常在一个循环中使用，效果和调用行遍历forEachRow()差不多。比如：

``` swift
var results = [[String?]]()
while let row = results?.next() {
	results.append(row)
}
```

### query 查询

``` swift
public func query(statement stmt: String) -> Bool
```

将给定字符串作为SQL语句进行查询。

### storeResults 保存查询结果

``` swift
public func storeResults() -> MySQL.Results?
```

该操作从服务器上获取一个完整的结果记录集并保存到客户机上。调用该函数应该在查询完毕后，在使用如forEachRow() 或 next()这样的遍历函数之前使用，这样就能够确保能够遍历到一个完整的查询结果记录集。

### setOption 设置选项

``` swift
public func setOption(_ option: MySQLOpt) -> Bool
public func setOption(_ option: MySQLOpt, _ b: Bool) -> Bool
public func setOption(_ option: MySQLOpt, _ i: Int) -> Bool
public func setOption(_ option: MySQLOpt, _ s: String) -> Bool
```

为当前连接设置选项。如果设置成功，则返回布尔类型的真值，否则为假。不同版本的MySQLOpt选项会有一些差别，选项值可能是布尔类型、整型或者字符串。

MySQLOpt的选项值可以参考以下枚举定义：

``` swift
public enum MySQLOpt {
    case MYSQL_OPT_CONNECT_TIMEOUT, MYSQL_OPT_COMPRESS, MYSQL_OPT_NAMED_PIPE,
        MYSQL_INIT_COMMAND, MYSQL_READ_DEFAULT_FILE, MYSQL_READ_DEFAULT_GROUP,
        MYSQL_SET_CHARSET_DIR, MYSQL_SET_CHARSET_NAME, MYSQL_OPT_LOCAL_INFILE,
        MYSQL_OPT_PROTOCOL, MYSQL_SHARED_MEMORY_BASE_NAME, MYSQL_OPT_READ_TIMEOUT,
        MYSQL_OPT_WRITE_TIMEOUT, MYSQL_OPT_USE_RESULT,
        MYSQL_OPT_USE_REMOTE_CONNECTION, MYSQL_OPT_USE_EMBEDDED_CONNECTION,
        MYSQL_OPT_GUESS_CONNECTION, MYSQL_SET_CLIENT_IP, MYSQL_SECURE_AUTH,
        MYSQL_REPORT_DATA_TRUNCATION, MYSQL_OPT_RECONNECT,
        MYSQL_OPT_SSL_VERIFY_SERVER_CERT, MYSQL_PLUGIN_DIR, MYSQL_DEFAULT_AUTH,
        MYSQL_OPT_BIND,
        MYSQL_OPT_SSL_KEY, MYSQL_OPT_SSL_CERT,
        MYSQL_OPT_SSL_CA, MYSQL_OPT_SSL_CAPATH, MYSQL_OPT_SSL_CIPHER,
        MYSQL_OPT_SSL_CRL, MYSQL_OPT_SSL_CRLPATH,
        MYSQL_OPT_CONNECT_ATTR_RESET, MYSQL_OPT_CONNECT_ATTR_ADD,
        MYSQL_OPT_CONNECT_ATTR_DELETE,
        MYSQL_SERVER_PUBLIC_KEY,
        MYSQL_ENABLE_CLEARTEXT_PLUGIN,
        MYSQL_OPT_CAN_HANDLE_EXPIRED_PASSWORDS
}
```

## MySQL Results API

结果记录集程序API提供了对查询结果进行操作的方法。

### close

``` swift
public func close()
```

关闭结果记录集并释放存储资源。请在完成每个数据库会话之后确保关闭操作，否则可能引发内存问题。数据库关闭通常可以用defer块的方式进行滞后操作，如前例所示。

### dataSeek 根据行位置检索数据

``` swift
public func dataSeek(_ offset: UInt)
```

根据指定的偏移量作为参数，将数据行指针指向结果记录集的目标行。

### numRows数据行统计

``` swift
public func numRows() -> Int
```

该方法能够获取结果记录集内所包含的数据行总数。

### numFields数据列统计

``` swift
public func numFields() -> Int
```

与numRows方法类似，但是返回的是当前结果记录集包含多少列数据。对于形如`Select *`的SQL查询来说非常有用，因为在查询之前可能不知道结果记录集究竟包含多少个字段。

### next返回下一行

``` swift
public func next() -> Element?
```

如果结果记录集内当前行指针指向位置存在下一行时，返回到下一行。

### forEachRow结果记录集的行遍历

``` swift
public func forEachRow(callback: (Element) -> ())
```

在查询结果记录集内遍历所有行。对于用数组或者字典追加数据元素时特别有用，详见[快速上手](#快速上手)。

### MySQL Statements API

### init 构造函数

``` swift
public init(_ mysql: MySQL)
```

初始化MySQL语句结构。通常用法是在用字符串作为参数传入查询语句后创建这样一个查询结构，便于后续其它API接口函数使用。

### close 关闭结构

``` swift
public func close()
```

该函数释放MySQL语句结构指针所占用的资源。请务必在应用结束后调用该语句，否则会消耗MySQL C API函数库的宝贵内存。通常在defer块内滞后执行，如前例[快速上手](#快速上手)所示。

### ping 检查连接

MySQL 在长时间待机时有可能超时断线，用 `ping()` 函数可以检测连接并根据需要自动重连。 

``` swift
guard mysql.ping() else {
	// 彻底断线了
}
```

### reset 复位重置

``` swift
public func reset()
```

重置服务器端的查询语句缓冲区。这个操作不会影响变量绑定和已存储的查询结果记录集。详见[MySQL语句重置命令参考（英文）](https://dev.mysql.com/doc/refman/5.7/en/mysql-stmt-reset.html)。

### clearBinds清除变量绑定

``` swift
func clearBinds()
```

清除当前的变量绑定。

### freeResult释放结果记录集

``` swift
public func freeResult()
```

释放由SQL查询或者编译SQL语句产生的结果记录集。调用该方法时，如果编译语句中有活动的记录游标，则连同游标一并关闭。

### errorCode & errorMessage 错误代码和错误消息

``` swift
public func errorCode() -> UInt32
```

``` swift
public func errorMessage() -> String
```

错误代码和错误消息在调试程序过程中作用巨大。详细内容请参考[MySQL服务器错误代码和错误消息](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html)。连接数据库后即可使用，举例如下：

``` swift
let mysql = MySQL()
let connected = mysql.connect(host: dbHost, user: dbUser, password: dbPassword, db: dbName)
guard connected else {
	// 验证是否连接成功
	print(mysql.errorMessage())
	return
}
```

在上面的例子里，如果连接失败则控制台会输出错误消息。

### prepare 准备SQL语句

``` swift
public func prepare(statement query: String) -> Bool
```

为执行查询准备SQL语句。通常该方法是在本工具集的其它程序接口API调用，但如果需要您也可以直接调用。

### execute 执行SQL查询

``` swift
public func execute() -> Bool
```

执行一个已经准备好的SQL查询语句

### results 结果记录集

``` swift
public func results() -> MySQLStmt.Results
```

从服务器返回当前查询的结果记录集。

### fetch 取出数据记录

``` swift
public func fetch() -> FetchResult
```

从结果记录集内取出下一行数据。

### numRows结果记录集行数统计

``` swift
public func numRows() -> UInt
```

返回结果缓冲记录集内有多少行记录。

### affectedRows受影响的数据行统计

``` swift
public func affectedRows() -> UInt
```

当采取UPDATE、DELETE或者INSERT语句时，数据表格发生内容更新、删除或增加，此时调用该函数则会得到数据表内有多少条记录因此改变。

### insertId 插入行ID号

``` swift
public func insertId() -> UInt
```

如果数据表存在自动递增的字段（称为ID），则返回最后一条插入行的ID号。

### fieldCount字段统计

``` swift
public func fieldCount() -> UInt
```

返回最后一次查询的结果记录集内的数据列总数，即包括多少个字段

### nextResult下一个结果记录集

``` swift
public func nextResult() -> Int
```

如果查询后存在多个结果记录集的情况下，返回下一个结果记录集。

### dataSeek选择特定记录行

``` swift
public func dataSeek(offset: Int)
```

调用该方法将按指定的行偏移量，在结果记录集内选择指定的记录行。

### paramCount参数统计

``` swift
public func paramCount() -> Int
```

返回一个已准备好的SQL语句内的参数总数。

### bindParam绑定参数

``` swift
public func bindParam()
func bindParam(_ s: String, type: enum_field_types)
public func bindParam(_ d: Double)
public func bindParam(_ i: Int)
public func bindParam(_ i: UInt64)
public func bindParam(_ s: String)
public func bindParam(_ b: UnsafePointer<Int8>, length: Int)
public func bindParam(_ b: [UInt8])
```

以上不同变量形式的绑定参数语句bindParam()统一了将不同变量绑定到查询参数的调用方法。如果调用时不含任何参数，则创建一个空绑定。
