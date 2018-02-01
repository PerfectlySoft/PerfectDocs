# SQLite

SQLite连接库提供了对SQLite3的使用封装，允许您的Perfect应用程序于SQLite数据库进行交互操作。

### 系统要求

### macOS

不需要额外的配置工作，SQlite3已经内置预装。

### Linux
确定您已经运行了SQLite3：
`sudo apt-get install sqlite3`

## 设置

在您的Package.swift文件中增加对"Perfect-SQLite"的库函数依存关系：

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-SQLite.git",
	majorVersion: 3
	)
```

### 声明和导入

为了使用SQLite函数库，首先需要在您开发的源程序开始部分增加声明和导入操作：

``` swift
import PerfectSQLite
```

### 快速上手

### 连接数据库

数据库的访问是通过其逻辑文件路径实现的，因此第一步是确定SQLite数据库的保存位置

``` swift
let dbPath = "./db/database"
```

一旦设置完成，即可打开连接并采用do-try-catch方式确保所有错误都在掌控之中：

``` swift
let dbPath = "./db/database"

do {
    let sqlite = try SQLite(dbPath)
    defer {
        sqlite.close() // This makes sure we close our connection.
    }
} catch {
    //Handle Errors
}
```

### 创建数据表

一旦完成上述的数据库连接，我们就可以通过[execute](#execute) 方法执行创建数据表的操作：

``` swift
let dbPath = "./db/database"

do {
    let sqlite = try SQLite(dbPath)
    defer {
        sqlite.close() // 此处确定我们关闭了数据库连接
    }

    try sqlite.execute(statement: "CREATE TABLE IF NOT EXISTS demo (id INTEGER PRIMARY KEY NOT NULL, option TEXT NOT NULL, value TEXT)")

} catch {
    //处理错误
}
```

### 运行一个或多个查询

* 请注意，Swift语言的字符串内变量值替换在此无效。如果需要使用变量，您需要使用绑定系统方法，详见[下一节](#将变量绑定到查询语句中)。

一旦完成数据库设置连接和数据表创建，下一步就可以进行查询并返回数据。以下的示例，我们将查询语句保存到一个字符串，然后作为参数传递给[forEachRow](#foreachrow-with-handlerow)行遍历方法，通常您可以在此将遍历后的数据结果追加到一个字典对象上去。

``` swift
let dbPath = "./db/database"
var contentDict = [String: Any]()

do {
	let sqlite = try SQLite(dbPath)
	defer {
		sqlite.close() // 确保数据库连接已关闭
	}

	let demoStatement = "SELECT * FROM demo"

	try sqlite.forEachRow(statement: demoStatement) {(statement: SQLiteStmt, i:Int) -> () in

		self.contentDict.append([
			"id": statement.columnText(position: 0),
			"second_field": statement.columnText(position: 1),
			"third_field": statement.columnText(position: 2)
		])
	}
} catch {
    //处理错误
}
```

### 将变量绑定到查询语句中

相信您肯定希望将查询中增加变量内容。如前所述，Swift语言自带的字符串变量值自动替换失效，此时需要手工绑定数据系统，如下所示：

``` swift
let dbPath = "./db/database"
var contentDict = [String: Any]()

do {
	let sqlite = try SQLite(dbPath)
	defer {
		sqlite.close() // 此处确定关闭数据连接
	}

	let demoStatement = "SELECT post_title, post_content FROM posts ORDER BY id DESC LIMIT :1"

	try sqlite.forEachRow(statement: demoStatement, doBindings: {
		(statement: SQLiteStmt) -> () in

		let bindValue = 5
		try statement.bind(position: 1, bindValue)

    }) {(statement: SQLiteStmt, i:Int) -> () in

		self.contentDict.append([
			"id": statement.columnText(position: 0),
			"second_field": statement.columnText(position: 1),
			"third_field": statement.columnText(position: 2)
		])
  	}
} catch {
    // 错误处理
}
```

是不是看起来技巧性过高了？没关系。我们提供的"doBindings:" 参数可以将一个闭包传递给行遍历循环，对您指定的字段位置进行变量绑定。并且我们最后的一个参数i（技术上来说称为“行句柄”）被忽略了，因为作为参数传递的闭包已经替代了这个句柄，这也是使用Swift的标准方式。相信您多用几次就会习惯。

## 完整的SQLite对象API参考

该对象所有的函数API包括：

``` swift
init(_:readOnly:)
close()
prepare(_:)
lastInsertRowID()
totalChanges()
changes()
errCode()
errMsg()
execute(_:)
execute(_:doBindings:)
execute(_:count:doBindings:)
doWithTransaction(_:)
forEachRow(_:handleRow:)
forEachRow(_:doBindings:handleRow:)
```

依次解释如下

### init初始化数据库

``` swift
public init(_ path: String, readOnly: Bool = false) throws
```

该类的基本构造函数，在创建新实例时调用。初始化过程中将数据库文件所在路径以参数传递给对象；而如有必要的话，第二个参数是用于确保数据库处于只读模式。调用该构造函数会抛出异常，因此在使用时必须要用do-try-catch进行错误处理。

### close关闭数据连接

``` swift
public func close()
```

即关闭数据库连接。默认的使用方式是在初始化数据库时的do-try-catch包内使用。这样的做法可以确保无论是连接成功还是出现错误而导致失败的情况下，数据库的连接都能够正常关闭。

### prepare 准备查询

``` swift
public func prepare(stat: String) throws -> SQLiteStmt
```

该函数返回一个编译好的SQLite语句对象。其它很多类函数（比如execute执行查询和forEachRow行遍历）都依赖于这个编译后的语句对象。这些函数会把查询语句以字符串变量的方式传递给函数，然后使用返回的对象句柄来实现与数据库通信。一般很少会在程序中直接使用这个函数，但是相信您也可以随时根据需要进行相应的调用。

### lastInsertRowID 返回最后一次插入的数据行ID号

``` swift
public func lastInsertRowID() -> Int
```

该函数返回最后一次插入数据库的所在行的值内容。使用时要额外注意，该语句必须在您的数据库初始化阶段do-try-catch当中创建数据连接后*在所有其它数据操作之前*调用，否则总会返回一个“0”。而“0”值则意味着要么数据库未连接，要么数据表内没有数据。如果目标的数据表被其它不同的线程进行数据行插入的操作，则调用这个函数的结果会很难预料。因此需要小心使用这个函数。详情请见[SQLite3 文档（英文）](https://www.sqlite.org/c3ref/last_insert_rowid.html) 用于理解这个函数的工作机制。

### totalChanges全部变化

``` swift
public func totalChanges() -> Int
```

返回sqlite3 *全部收到影响的行总数* 。详见[SQLite3 文档（英文版）](https://www.sqlite.org/c3ref/total_changes.html):

*“该函数返回自创建数据库连接后，由所有INSERT、UPDATE或DELETE语句完成后影响到的所有数据行总数，包括触发器。其它的SQL查询语句不会影响到该总数统计”*

一旦关闭数据连接，则这个值将变得无法访问，因此请确保您的调用是在do-try-catch保护之下执行有关查询。

### changes 变化

``` swift
public func changes() -> Int
```

该函数返回sqlite3_changes所说明的数据库变化值。与totalChanges最大的区别在于，该函数只会返回与当前查询语句有关的变化值，不会包括如触发器等连锁引发的行数。

### errCode 错误代码

``` swift
public func errCode() -> Int
```

返回sqlite3_errcode错误代码。详细参考请[查看这里（英文）](https://www.sqlite.org/rescode.html)。

### errMsg 错误详细信息

``` swift
public func errMsg() -> String
```

返回sqlite3_errmsg错误信息。详细参考请[查看这里（英文）](https://www.sqlite.org/c3ref/errcode.html)。

### execute

``` swift
public func execute(statement: String) throws
```

执行查询并无需返回记录集数据（比如 *INSERT* 或 *CREATE TABLE* ）。由于该函数会抛出错误，因此请在您的do-try-catch中对可能发生的错误进行准备和处理

### execute 及 doBindings 参数

``` swift
public func execute(statement: String, doBindings: (SQLiteStmt) throws -> ()) throws
```

在查询之前可以将变量内容绑定到查询语句中去。同样因抛出异常的缘故，请在do-try-catch语句块中使用。详情请参考[变量绑定](#将变量绑定到查询语句中)。

### execute及count: 和doBindings:参数

``` swift
public func execute(statement: String, count: Int, doBindings: (SQLiteStmt, Int) throws -> ()) throws
```

最后一个变量用于指定同一个查询重复的次数。这个参数在您需要循环执行一个插入语句时特别有用。同样允许[变量绑定](#将变量绑定到查询语句中)。必须进行do-try-catch异常处理。

### doWithTransaction 事务处理

``` swift
public func doWithTransaction(closure: () throws -> ()) throws
```

“自BEGIN开始执行，调用指定的闭包；如果执行失败或出现错误，则进行ROLLBACK回滚处理；如果操作成功没有异常发生则用COMMIT提交并结束事务”

这意味者运行闭包后可以返回一个结果，如果结果失败的话，那么数据库其实不会有数据改变；只有返回成功才证明整个事务处理完成。详见[SQLite3 事务处理](https://www.sqlite.org/lang_transaction.html)。

### forEachRow with handleRow

``` swift
public func forEachRow(statement: String, handleRow: (SQLiteStmt, Int) -> ()) throws
```

执行给定的查询语句，返回数据结果后，根据指定的闭包进行逐行遍历，遍历过程中可以通过行句柄handleRow加以区分每一个数据行。这种方式对于将查询结果记录集转化为字典或者数组时特别有用。由于会抛出异常，因此必须在您的do-try-catch中处理异常。

### forEachRow 带 doBindings: 变量绑定和 handleRow: 行句柄

``` swift
public func forEachRow(statement: String, doBindings: (SQLiteStmt) throws -> (), handleRow: (SQLiteStmt, Int) -> ()) throws
```

与上一个函数类似，该函数允许在遍历结果记录集内每一行数据的过程中执行闭包，但包括了在运行查询之前的变量绑定。由于会抛出异常，因此必须在您的do-try-catch中处理异常。

## 示例

以下的例子是一个博客系统的程序组成部分。这个函数封装了博客从SQLite数据库中加载每个博客内容并追加到一个声明为`var content = [String: Any]()`的类对象。这个例子在mustache模板中经常被使用。在示例中，每页会加载5篇博文，并将页码作为参数传递到函数中去：

``` swift
func loadPageContent(forPage: Int) {
	do {
		let sqlite = try SQLite(DB_PATH)
		defer {
			sqlite.close()  // 在完成数据请求后自动关闭数据库连接
		}
		let sqlStatement = "SELECT post_content, post_title FROM posts ORDER BY id DESC LIMIT 5 OFFSET :1"
	
		try sqlite.forEachRow(statement: sqlStatement, doBindings: {
			(statement: SQLiteStmt) -> () in
			
			let bindPage: Int
			if self.page == 0 || self.page == 1 {
				bindPage = 0
			} else {
				bindPage = forPage * 5 - 5
			}

			try statement.bind(position: 1, bindPage)
      		}) {
				(statement: SQLiteStmt, i:Int) -> () in
					self.content.append([
						"postContent": statement.columnText(position: 0),
						"postTitle": statement.columnText(position: 1)
					])
			}
		} catch {
	}
}
```
