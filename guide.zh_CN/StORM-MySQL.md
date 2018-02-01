# MySQLStORM

### 在工程中引用

请首先在您的 Package.swift 文件中更新依存关系，以确保必要的函数库能够下载到编译环境用于数据库连接：

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM", majorVersion: 3)
```

### 关于防范SQL注入式攻击

StORM 并无法确保您的数据被黑客通过参数化捆绑方式进行SQL注入式攻击。

但是在某些函数中，您是完全可以验证SQL查询的语句合法性的，因此，请务必在使用SQL时注意检查外来数据内容。

## 创建数据库连接

为了创建数据库连接，请自行更新数据源配置：

``` swift
MySQLConnector.host        = "localhost"
MySQLConnector.username    = "username"
MySQLConnector.password    = "secret"
MySQLConnector.database    = "yourdatabase"
MySQLConnector.port        = 3306
```

一旦连接参数配置完成，则数据库会在需要时自动连接。


``` swift
let obj = User()
```

## MySQLStORM 支持的函数方法

### 数据库连接

`MySQLConnector ` - 设置MySQL 服务器连接参数，包括主机名，端口，用户名和密码，以及默认数据库等等。

### 创建数据表

`setup()` - 通过检查对象直接创建数据表。所有的数据字段会根据该对象的属性自动创建。如果属性以下划线或者 `internal_`命名，则该属性不会作为数据字段进行创建，会被忽略。

> **⚠️注意⚠️** 其中数据库的 **主索引键** 是该类中的第一个字段。

#### 为创建表格单独设置SQL语句

`setup(String)` - 手工设置创建数据表格的SQL语句，覆盖自动创建数据表的方法。

### 将程序中的对象数据保存到数据库

`save()` - 保存对象。如果当前没有设置主索引键的值，那么就会向数据库内插入一条新的数据。否则如果主索引键已经设置了具体的数值，则会进行一个更新操作。

`save {id in ... }` - 保存对象。如果当前没有设置主索引键的值，那么就会向数据库内插入一条新的数据。否则如果主索引键已经设置了具体的数值，则会进行一个更新操作。闭包用于如果创建成功的话，返回一个创建完成的数据行id。

`create()` - 显式要求以插入新数据的方式保存数据，无论主索引键是否已经设置了内容。

### 获取数据

`findAll()` - 获取全部数据行，仅受限于数据游标（9,999,999行）

`get()` - 假设主索引键已经设置的情况下，获取一行数据记录。

`get(Any)` - 根据主索引编号获取一行数据记录。

`find([(String, Any)])` - 根据查询标准进行检索。查询参数为关键词和对应值构成的字典。一旦匹配这个检查标准，数据记录就会被纳入查询结果。

`select(whereclause:	String,
		params:			[Any],
		orderby:		[String])` - 执行一个查询子句，捆绑参数以防范SQL注入式攻击。`orderby`是用于排序的一组关键词，每个字段都允许选择附加`ASC`或`DESC`，分别表示顺序排序和逆序排序。
		
此外，`select`子句可包括：

*  `cursor: StORMCursor` - 可选项 `cursor` 游标对象是一个 [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Cursor.md)

*  `columns: [String]` - 可选项 `columns` 是一个字段数组，用于说明查询结果需要返回的各个字段。如果这一个选项不进行设置，那么就会默认返回该数据表的所有字段。

### 删除数据记录

`delete()` - 删除当前对象对应的数据记录行，假设该行数据的主索引编号已经设置。

`delete(Any)` - 根据主索引编号删除对象及其对应的数据库记录。

`delete(Int, idname)` - 删除指定字段`idname`取值与`Int`相等的所有数据记录。

`delete(String, idname)` - 删除指定字段`idname`取值与`String`相等的所有数据记录。

`delete(UUID, idname)` - 删除指定字段`idname`取值与`UUID`相等的所有数据记录。

### 插入数据

`insert([(String, Any)])` - 将数据记录以字典的表达方式插入数据，每个字典条目的字符串表示字段名称，条目的取值就是目标的字段值。

`insert(cols: [String], params: [Any])` - 将数据记录以两列不同的数组的表达形式进行插入数据，第一个参数是字段名称构成的数组，第二个参数是具体对应每个字段在本条记录中的取值构成的另外一个数组。

`insert(cols: [String], params: [Any], idcolumn: String)` - 同上，不过要求返回idcolumn 字段值作为查询结果。

### 更新数据

`update(data: [(String, Any)], idName: String = "id", idValue: Any)` - 根据主索引编号更新有关数据记录，以字典方式表达数据（每个字典条目对应一个字段名称及该字段的取值），包括可选idName字段名称。

`update(cols: [String], params: [Any], idName: String, idValue: Any)` - 根据主索引编号更新有关数据记录，以双数组方式表达数据（cols数组是字段名，params数组是对应字段值），包括可选idName字段名称。

⚠️译者注⚠️可选的字段名称idName可以是主索引字段，也可以是其他字段，根据程序员选择而定。

### Upserts

`upsert(cols: [String], params: [Any], conflictkeys: [String])` - 插入指定数据；如果发现有重复记录（用`conflictkeys`标出的字段列为不可重复）的情况下，执行更新（覆盖）。

### SQL 语句

`sqlExec(String)` - 直接执行SQL查询。

`sql(String, params: [String])` - 执行SQL查询，附带绑定参数数组。查询完成后会返回一个[PGResult]查询结果数组。

`sqlAny(String, params: [String])` 执行SQL查询，附带绑定参数数组。查询完成后会返回一个[StORMRow]查询结果数组。

`sqlRows(String, params: [String])` - 执行SQL查询，附带绑定参数数组。查询完成后会返回一个[StORMRow]查询结果数组。
