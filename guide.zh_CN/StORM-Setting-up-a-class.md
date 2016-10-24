# 使用 StORM 数据类

首先请将函数库引入程序：

``` swift
import PostgresStORM
// 如果您用的其他数据源，比如SQLite，请这样声明：
import SQLiteStORM
```

当您在 StORM 基础上创建数据类的时候，会触发数据库自动构造一个表格，使得与数据库操作实现自动化。

如果您使用的是 PostgreSQL，请以 PostgresStORM 作为基类声明：

``` swift
class User: PostgresStORM {
}
```

如果您使用的是 SQLite，请以 SQLiteStORM 作为基类声明：

``` swift
class User: SQLiteStORM {
}
```

下一步，请为您的类增加属性，该操作将自动更细相关的数据表结构。在此强烈建议在编写属性部分程序的时候，请不要使用非标准字符，因为可能数据库不接受这样的字段，比如“🍒”。

*⚠️注意⚠️* 该对象的第一个属性将成为对应数据表的主索引 —— 传统的方式就是给主索引列起名叫做 `id`，虽然您可以为主索引字段设置任何有效的名字。SQL这种关系数据库的主索引典型类型是整型、字符串或者UUID编码。如果您的主索引不是自动递增的整数，则一定要设置好这个id值，以保证数据的完整性和一致性。

``` swift
// ⚠️注意⚠️：第一个属性将成为主索引字段，所以应该是ID。
var id				: Int = 0
var firstname		: String = ""
var lastname		: String = ""
var email			: String = ""
```

您可能已经注意到上面的程序通过给每个字段设置默认值，而不是在`init()`构造函数中设置。这样看起来更简单一些。如果您选择自行编写`init()` 或 `init(_ connect: XXConnect)` 构造函数，请注意一定要在构造函数中首先调用基类的构造函数`super.init()`，而且要小心数据库的连接属性。

### 确定数据表格

为了避免数据库内的命名冲突，请用下列方法自行设置每个数据类的对应数据表名称，如下所示：

``` swift
override open func table() -> String {
	return "users"
}
```

### 数据类对象与数据库的属性绑定

为了能够让新编写的数据类正常工作，比如查询数据并把结果记录集转换成为对象数组等等，请一定要在您的数据类中增加以下两个函数：

``` swift
override func to(_ this: StORMRow) {
	id				= this.data["id"] as! Int
	firstname		= this.data["firstname"] as! String
	lastname		= this.data["lastname"] as! String
	email			= this.data["email"] as! String
}

func rows() -> [User] {
	var rows = [User]()
	for i in 0..<self.results.rows.count {
		let row = User()
		row.to(self.results.rows[i])
		rows.append(row)
	}
	return rows
}
```

特别注意在 `to` （绑定到）函数中，要逐个指定所有属性。
这种处理方法的好处是能够在这个阶段验证绑定操作是否有效，您还可以在这个过程中增加更多自定义操作。

而`rows` （数据行）函数则是用于处理简化结果记录集为对象数组。请注意在使用上面的代码时，⚠️一定要⚠️把第三行的对象类名称改过来。


### 完整案例

以下是整个类的完整例子：

``` swift
import PostgresStORM

class User: PostgresStORM {
	// NOTE: 注意，第一个属性将会是数据表主索引。
	var id				: Int = 0
	var firstname		: String = ""
	var lastname		: String = ""
	var email			: String = ""


	override open func table() -> String {
		return "users"
	}

	override func to(_ this: StORMRow) {
		id				= this.data["id"] as! Int
		firstname		= this.data["firstname"] as! String
		lastname		= this.data["lastname"] as! String
		email			= this.data["email"] as! String
	}

	func rows() -> [User] {
		var rows = [User]()
		for i in 0..<self.results.rows.count {
			let row = User()
			row.to(self.results.rows[i])
			rows.append(row)
		}
		return rows
	}
}
```
