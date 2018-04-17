# Perfect CRUD

CRUD 是一个基于Swift 4以上版本的关系数据库对象管理系统（ORM）。该函数库采用Swift 4 `Codable` （可编码）协议映射到SQL类型的数据库表格。CRUD能够创建基于`Codable`类型的数据结构并映射到同构数据表格中，实现数据记录插入、更新之类的典型操作。CRUD同样可以进行数据库查询和表格视图合并的操作，所有这些操作都可确保数据类型安全。

CRUD 采用了一种简洁明了但是又表达形式丰富多样、数据类型在编译阶段得到安全检查保障的方法来构造查询操作，其实现目标是轻量化、免依存关系。实现方法采用了类型模板（Generics）、字段路径（KeyPath）和可编码（Codable）以确保编译阶段对上述内容的一致检查。

目前可以使用的实际数据实现可以在这里找到：[SQLite](https://github.com/PerfectlySoft/Perfect-SQLite)、[Postgres](https://github.com/PerfectlySoft/Perfect-PostgreSQL)和[MySQL](https://github.com/PerfectlySoft/Perfect-MySQL)。

# Contents
* <a href="#general-usage">General Usage</a>
* <a href="#operations">Operations</a>
	* <a href="#database">Database</a>
		* <a href="#transaction">Transaction</a>
		* <a href="#create">Create</a>
			* <a href="#create-policy">Policy</a>
		* <a href="#table">Table</a>
		* <a href="#sql">SQL</a>
	* <a href="#table-1">Table</a>
	* <a href="#join">Join</a>
		* <a href="#parent-child">Parent Child</a>
		* <a href="#many-to-many">Many to Many</a>
		* <a href="#self-join">Self Join</a>
		* <a href="#junction-join">Junction Join</a>
	* <a href="#where">Where</a>
		* <a href="#expression-operators">Expression Operators</a>
	* <a href="#order">Order</a>
	* <a href="#limit">Limit</a>
	* <a href="#update">Update</a>
	* <a href="#insert">Insert</a>
	* <a href="#delete">Delete</a>
	* <a href="#select">Select</a>
	* <a href="#count">Count</a>
* <a href="#codable-types">Codable Types</a>
	* <a href="#identity">Identity</a>
* <a href="#error-handling">Error Handling</a>
* <a href="#logging">Logging</a>

<a name="general-usage"></a>

## General Usage

以下是CRUD的简明实用方法

```swift
// CRUD 可以应用到大多数可编码类型：
struct PhoneNumber: Codable {
	let personId: UUID
	let planetCode: Int
	let number: String
}
struct Person: Codable {
	let id: UUID
	let firstName: String
	let lastName: String
	let phoneNumbers: [PhoneNumber]?
}

// 使用CRUD首先需要创建数据库连接，创建数据连接的方法根据具体数据库客户端有所不同。
// 首先创建一个 `Database` 对象并进行配置，以下示范代码采用了SQLite作为演示。
// 但是所有代码对于各种其他数据库来说应该都是一样的

let db = Database(configuration: try SQLiteDatabaseConfiguration(testDBName))

// 如果数据库目前还不存在这个表格，则自动生成
// 数据表的创建过程默认是递归的，比如如果一个数据结构包含了子表“PhoneNumber”，
// 则子表同样会一并创建。
try db.create(Person.self, policy: .reconcileTable)

// 获得表格对象参考，用于后续操作如插入数据
let personTable = db.table(Person.self)
let numbersTable = db.table(PhoneNumber.self)

// 如果索引不存在，则自动为personId追加一个索引
try numbersTable.index(\.personId)

// 插入样本数据
do {
	// 插入样本数据
	let owen = Person(id: UUID(), firstName: "Owen", lastName: "Lars", phoneNumbers: nil)
	let beru = Person(id: UUID(), firstName: "Beru", lastName: "Lars", phoneNumbers: nil)
	
	// 插入人员记录
	try personTable.insert([owen, beru])
	
	// 设置电话好吗
	try numbersTable.insert([
		PhoneNumber(personId: owen.id, planetCode: 12, number: "555-555-1212"),
		PhoneNumber(personId: owen.id, planetCode: 15, number: "555-555-2222"),
		PhoneNumber(personId: beru.id, planetCode: 12, number: "555-555-1212")])
}

// 执行查询
// 找到所有姓氏为Lars 并且电话区号为12的人。
let query = try personTable
		.order(by: \.lastName, \.firstName)
	.join(\.phoneNumbers, on: \.id, equals: \.personId)
		.order(descending: \.planetCode)
	.where(\Person.lastName == "Lars" && \PhoneNumber.planetCode == 12)
	.select()

// 遍历结果集并打印名单
for user in query {
	// We joined PhoneNumbers, so we should have values here.
	guard let numbers = user.phoneNumbers else {
		continue
	}
	for number in numbers {
		print(number.number)
	}
}
```

<a name="operations"></a>
## Operations

CRUD 的主要操作集中于数据库对象连接之后进行一系列的数据库操作。某些操作是立刻执行的，而另外一些操作（比如select查询）是根据需要才实际执行。每个操作的结果都是可以继续作为参考进行下一步操作，即链式操作。

以下操作介绍的分类是依据实现相应操作的有关对象。注意为了便于示范，很多类型定义都是缩写，而且部分函数以扩展方式实现，以确保整体性集中。

<a name="database"></a>
### Database

数据库对象用于封装数据库连接，通过 `DatabaseConfigurationProtocol` 协议对象实现，并根据具体数据库实现有所不同：
 
```swift
// postgres 配置范例
let db = Database(configuration: 
	try PostgresDatabaseConfiguration(database: postgresTestDBName, host: "localhost"))
	
// sqlite 配置范例
let db = Database(configuration: 
	try SQLiteDatabaseConfiguration(testDBName))
```

数据库对象实现了以下逻辑函数：

```swift
public struct Database<C: DatabaseConfigurationProtocol>: DatabaseProtocol {
	public typealias Configuration = C
	public let configuration: Configuration
	public init(configuration c: Configuration)
	public func table<T: Codable>(_ form: T.Type) -> Table<T, Database<C>>
	public func transaction<T>(_ body: () throws -> T) throws -> T
	public func create<A: Codable>(_ type: A.Type, 
		primaryKey: PartialKeyPath<A>? = nil, 
		policy: TableCreatePolicy = .defaultPolicy) throws -> Create<A, Self>
}
```

数据库对象的操作包括事务 `transaction`、表格创建 `create`和表格引用 `table`。

<a name="transaction"></a>
#### Transaction

数据库事务操作 `transaction` 会执行在 “BEGIN“和”COMMIT“（或 “ROLLBACK”） 之间的系列子操作。如果这个系列操作中间没有抛出错误，则事务提交成功，否则会回滚。

```swift
public extension Database {
	func transaction<T>(_ body: () throws -> T) throws -> T
}
```

比如：

```swift
try db.transaction {
	... 系列操作
}
```

事务操作的尾随闭包允许带返回值

```swift
let value = try db.transaction {
	... further operations
	return 42
}
```

<a name="create"></a>
#### Create

表格创建的操作必须通过一个可编码的数据结构实例化完成。在具备条件的情况下会将实例的结构创建同构数据表。表格主索引可以在创建中声明，也可以通过“表格创建规则”用于隐式说明。

```swift
public extension DatabaseProtocol {
	func create<A: Codable>(
		_ type: A.Type, 
		primaryKey: PartialKeyPath<A>? = nil, 
		policy: TableCreatePolicy = .defaultPolicy) throws -> Create<A, Self>
}
```

举例：

```swift
try db.create(TestTable1.self, primaryKey: \.id, policy: .reconcileTable)
```

<a name="create-policy">`TableCreatePolicy`</a> 表格创建规则包括下列选项：

* .reconcileTable（求同除异）- 如果数据库存在同名表格但字段有所不同，则现存表格的字段将根据当前新的定义进行调整，即如果包含新字段，则增加新字段；如果表内存在未定义的字段，则未定义的旧字段将被删除。唯一注意的是，如果旧表存在同名字段但是类型不一致，则该旧字段会保持不动。比如新字段类型为字符串而旧字段为整型变量，则这种变更不会发生。
* .shallow （浅表创建）- 如果声明，则合并类型的表格不会自动创建；如果不声明，则所有合并类型的表格都会自动创建。
* .dropTable （先删后建）- 创建表格前先删除同名表格。这个方式在软件开发和测试阶段特别好用，或者对于某些只保存临时数据的表格也很适用，只要程序重启则旧表格会被自动覆盖。

只要不应用 `.dropTable` 和 `.reconcileTable` 规则，即使表格存在，调用表格创建操作实际上是安全的，现存表格结构和记录都不会修改。

<a name="table"></a>
#### Table

`table` 操作返回数据表的可编码对象，用于后续操作。

```swift
public protocol DatabaseProtocol {
	func table<T: Codable>(_ form: T.Type) -> Table<T, Self>
}
```

举例：

```swift
let table1 = db.table(TestTable1.self)
```

<a name="sql"></a>
#### SQL

CRUD 也可以直接执行SQL语句，返回结果为任何适当的可编码类型数组。

```swift
public extension Database {
	func sql(_ sql: String, bindings: Bindings = []) throws
	func sql<A: Codable>(_ sql: String, bindings: Bindings = [], _ type: A.Type) throws -> [A]
}
```

比如

```swift
try db.sql("SELECT * FROM mytable WHERE id = 2", TestTable1.self)
```

<a name="table-1"></a>
### Table

**Table** 可跟随：`Database`.

**Table** 支持：`update`, `insert`, `delete`, `join`, `order`, `limit`, `where`, `select`, and `count`.


数据表对象用于执行数据更新、插入、删除或者查询。表对象实例的获得必须通过数据库对象配以预定义可编码结构实现。数据表对象只能够出现在表数据相关操作中，而且必须在相关链式操作开始之前准备好。

表对象的参数化是根据程序代码提供的预定义可编码结构完成的，而数据表类型也决定了后续操作的所有结果类型，以下统称 *OverAllForm（统合类型）*。

举例

```swift
// 获取用于代表数据表1的对象
// 所有数据插入删除和更新操作都会影响到"TestTable1"
// 而查询操作会创建TestTable1表格对象的集合
let table1 = db.table(TestTable1.self)
```

上面的例子中测试表1即为统合类型。对于该对象的删除操作将直接转换为数据表删除；针对该对象的select查询操作则自动生成测试表1的结果记录集对象。

<a name="join"></a>
### Join

**Join** 可跟随： `table`, `order`, `limit`, 或另外一个 `join`.

**Join** 支持：`join`, `where`, `order`, `limit`, `select`, `count`.

合并操作能够将父子关系表格或者多对多关系的表格进行合并查询。

<a name="parent-child">Parent-child</a> 父子关系表格示范：

```swift
struct Parent: Codable {
	let id: Int
	let children: [Child]?
}
struct Child: Codable {
	let id: Int
	let parentId: Int
}
try db.transaction {
	try db.create(Parent.self, policy: [.shallow, .dropTable]).insert(
		Parent(id: 1, children: nil))
	try db.create(Child.self, policy: [.shallow, .dropTable]).insert(
		[Child(id: 1, parentId: 1),
		 Child(id: 2, parentId: 1),
		 Child(id: 3, parentId: 1)])
}
let join = try db.table(Parent.self)
	.join(\.children,
		  on: \.id,
		  equals: \.parentId)
	.where(\Parent.id == 1)
guard let parent = try join.first() else {
	return XCTFail("Failed to find parent id: 1")
}
guard let children = parent.children else {
	return XCTFail("Parent had no children")
}
XCTAssertEqual(3, children.count)
for child in children {
	XCTAssertEqual(child.parentId, parent.id)
}
```

上述范例将子对象整合到了父对象的 `.children` 属性中，其对象类型是一个数组 `[Child]?` 。当查询执行时，所有parentId值为1 的子表记录都会被整合到查询结果中，形成典型的父子关系。

<a name="many-to-many">Many-to-many</a> 多对多范例：

```swift
struct Student: Codable {
	let id: Int
	let classes: [Class]?
}
struct Class: Codable {
	let id: Int
	let students: [Student]?
}
struct StudentClasses: Codable {
	let studentId: Int
	let classId: Int
}
try db.transaction {
	try db.create(Student.self, policy: [.dropTable, .shallow]).insert(
		Student(id: 1, classes: nil))
	try db.create(Class.self, policy: [.dropTable, .shallow]).insert([
		Class(id: 1, students: nil),
		Class(id: 2, students: nil),
		Class(id: 3, students: nil)])
	try db.create(StudentClasses.self, policy: [.dropTable, .shallow]).insert([
		StudentClasses(studentId: 1, classId: 1),
		StudentClasses(studentId: 1, classId: 2),
		StudentClasses(studentId: 1, classId: 3)])
}
let join = try db.table(Student.self)
	.join(\.classes,
		  with: StudentClasses.self,
		  on: \.id,
		  equals: \.studentId,
		  and: \.id,
		  is: \.classId)
	.where(\Student.id == 1)
guard let student = try join.first() else {
	return XCTFail("Failed to find student id: 1")
}
guard let classes = student.classes else {
	return XCTFail("Student had no classes")
}
XCTAssertEqual(3, classes.count)
for aClass in classes {
	let join = try db.table(Class.self)
		.join(\.students,
			  with: StudentClasses.self,
			  on: \.id,
			  equals: \.classId,
			  and: \.id,
			  is: \.studentId)
		.where(\Class.id == aClass.id)
	guard let found = try join.first() else {
		XCTFail("Class with no students")
		continue
	}
	guard nil != found.students?.first(where: { $0.id == student.id }) else {
		XCTFail("Student not found in class")
		continue
	}
}
```

<a name="self-join">Self Join</a> 自联合范例：

```swift
struct Me: Codable {
	let id: Int
	let parentId: Int
	let mes: [Me]?
	init(id i: Int, parentId p: Int) {
		id = i
		parentId = p
		mes = nil
	}
}
try db.transaction {
	try db.create(Me.self, policy: .dropTable).insert([
		Me(id: 1, parentId: 0),
		Me(id: 2, parentId: 1),
		Me(id: 3, parentId: 1),
		Me(id: 4, parentId: 1),
		Me(id: 5, parentId: 1)])
}
let join = try db.table(Me.self)
	.join(\.mes, on: \.id, equals: \.parentId)
	.where(\Me.id == 1)
guard let me = try join.first() else {
	return XCTFail("Unable to find me.")
}
guard let mes = me.mes else {
	return XCTFail("Unable to find meesa.")
}
XCTAssertEqual(mes.count, 4)
```

<a name="junction-join">Junction Join</a> 多表联合：

```swift
struct Student: Codable {
	let id: Int
	let classes: [Class]?
	init(id i: Int) {
		id = i
		classes = nil
	}
}
struct Class: Codable {
	let id: Int
	let students: [Student]?
	init(id i: Int) {
		id = i
		students = nil
	}
}
struct StudentClasses: Codable {
	let studentId: Int
	let classId: Int
}
try db.transaction {
	try db.create(Student.self, policy: [.dropTable, .shallow]).insert(
		Student(id: 1))
	try db.create(Class.self, policy: [.dropTable, .shallow]).insert([
		Class(id: 1),
		Class(id: 2),
		Class(id: 3)])
	try db.create(StudentClasses.self, policy: [.dropTable, .shallow]).insert([
		StudentClasses(studentId: 1, classId: 1),
		StudentClasses(studentId: 1, classId: 2),
		StudentClasses(studentId: 1, classId: 3)])
}
let join = try db.table(Student.self)
	.join(\.classes,
		  with: StudentClasses.self,
		  on: \.id,
		  equals: \.studentId,
		  and: \.id,
		  is: \.classId)
	.where(\Student.id == 1)
guard let student = try join.first() else {
	return XCTFail("Failed to find student id: 1")
}
guard let classes = student.classes else {
	return XCTFail("Student had no classes")
}
XCTAssertEqual(3, classes.count)
```

合并操作目前不支持更新、插入或者删除（以及层叠删除、递归更新等等， 都不支持）。

合并协议包括两个公用函数，一个用于处理双表联合，一个用于多表联合（三个以上）。

```swift
public protocol JoinAble: TableProtocol {
	// standard join
	func join<NewType: Codable, KeyType: Equatable>(
		_ to: KeyPath<OverAllForm, [NewType]?>,
		on: KeyPath<OverAllForm, KeyType>,
		equals: KeyPath<NewType, KeyType>) throws -> Join<OverAllForm, Self, NewType, KeyType>
	// junction join
	func join<NewType: Codable, Pivot: Codable, FirstKeyType: Equatable, SecondKeyType: Equatable>(
		_ to: KeyPath<OverAllForm, [NewType]?>,
		with: Pivot.Type,
		on: KeyPath<OverAllForm, FirstKeyType>,
		equals: KeyPath<Pivot, FirstKeyType>,
		and: KeyPath<NewType, SecondKeyType>,
		is: KeyPath<Pivot, SecondKeyType>) throws -> JoinPivot<OverAllForm, Self, NewType, Pivot, FirstKeyType, SecondKeyType>
}
```

横向合并需要三个参数：

`to` - 指向统合类型属性的字段路径。该字段路径指向的应该是一个可选类型数组，其元素类型应该是非集成的可编码数据结构。该属性用于设置结果集对象。

`on` - 用于统合类型的主索引的字段路径（通常就是主数据表的主索引字段）。

`equals` - 等价于统合类型的“on“字段路径，用于关系数据库的外部索引。

多表联合（三个以上，含三个）需要六个参数。

`to` - 指向统合类型属性的字段路径。该字段路径指向的应该是一个可选类型数组，其元素类型应该是非集成的可编码数据结构。该属性用于设置结果集对象。

`with` - 联合后的表格类型。

`on` - 用于统合类型的主索引的字段路径（通常就是主数据表的主索引字段）。

`equals` - 等价于统合类型的“on“字段路径，用于关系数据库的外部索引。

`and` - 指向子表的字段路径，用于联合表关键字段（通常为主表主索引）

`is` - 联合表的字段路径，应等价于子表的 `and` 属性。

如果不特别声明，任何联合后的表格都会在同和对象查询完成之后设置为空。

如果声明引用了一个联合表但是没有实际联合对象查询结果，则统合对象的结果属性会被设置为一个空数组。

<a name="where"></a>
### Where

**Where** 可跟随：`table`, `join`, `order`.

**Where** 支持： `select`, `count`, `update` (when following `table`), `delete` (when following `table`).

`where` 操作用于定义数据结果的过滤条件，过滤数据的结果可以用于查询、更新或者删除。只有在进行查询/统计、更新或删除操作时，才可以使用Where子句（过滤器对象）。

```swift
public protocol WhereAble: TableProtocol {
	func `where`(_ expr: Expression) -> Where<OverAllForm, Self>
}
```

Where 操作虽然是可选的，但是每个链式操作只允许有一个 `where` 子句，并且必须置于整个链式操作的末端作为限制条件。

示范代码：

```swift
let table = db.table(TestTable1.self)
// insert a new object and then find it
let newOne = TestTable1(id: 2000, name: "New One", integer: 40)
try table.insert(newOne)
// search for this one object by id
let query = table.where(\TestTable1.id == newOne.id)
guard let foundNewOne = try query.first() else {
	...
}
```

<a name="expression-operators"></a>
`where` 操作的参数是一个 `CRUDBooleanExpression` 布尔表达式对象，由一下任何运算符构造：

标准 Swift 运算符：

• 等价：`==`, `!=`

• 比较： `<`, `<=`, `>`, `>=`

• 逻辑 `!`, `&&`, `||`

自定义运算符：

• 包含/不包含: `~`, `!~`

• 类似于： `%=%`, `=%`, `%=`, `%!=%`, `!=%`, `%!=` 

对于等价或者比较运算符，左边的运算单元必须为可编码类型的一个字段路径，而右边的运算单元可以是这些类型： Int, Double, String, [UInt8], Bool, UUID, 或者 Date。字段路径可以是可选属性值，此时右运算单元可以是 nil，用于匹配查询中的空与非空类型。

上述等价和比较运算符是类型安全的，也就意味着，比如不能将整数与字符串直接比较。而右运算单元必须于左边的字段路径类型匹配。这种方式是Swift语言的典型应用，应该不会有意外。

通过表格或者联合查询而来的结果可以应用上述表达式。如果在查询中使用了并非在此定义的其他路径字段类型，则可能会导致运行时错误发生。

在下列代码中：

```swift
table.where(\TestTable1.id > 20)
```

`\TestTable1.id` 是一个路径字段，指向该对象的整型 id。右运算单元 20 是一个常量，而大于号 `>` 将二者进行运算操作并产生一个 `CRUDBooleanExpression` 布尔表达式用于直接限定 `where` 自居或者用于其他更复杂的表达式。

逻辑运算符允许两个 `CRUDBooleanExpresssion`进行或与非操作 `and`、`or`和 `not`。具体方法是使用 Swift标准运算符： `&&`、 `||`和 `!` 

```swift
table.where(\TestTable1.id > 20 && 
	!(\TestTable1.name == "Me" || \TestTable1.name == "You"))
```

包含/不包含运算的右运算单元为数组。

```swift
table.where(\TestTable1.id ~ [2, 4])
table.where(\TestTable1.id !~ [2, 4])
```

上述例子展示了所有 TestTable1 对象中 id 值属于数组内元素，或者不属于数组内元素（第二个例子）。

Like （类似于）运算符只用于字符串操作，用于说明字符串以预期方式匹配或者包含，用于文本字段检索：

```swift
try table.where(\TestTable2.name %=% "me") // 名称内包含 `me`
try table.where(\TestTable2.name =% "me") // 名称以 `me` 开始
try table.where(\TestTable2.name %= "me") // 名称以 `me` 结束
try table.where(\TestTable2.name %!=% "me") // 名称内不包含 `me`
try table.where(\TestTable2.name !=% "me") // 名称不以 `me` 开始
try table.where(\TestTable2.name %!= "me") // 名称不以 `me` 结束
```

<a name="order"></a>
### Order

**Order** 可跟随： `table`, `join`.

**Order** 支持： `join`, `where`, `order`, `limit` `select`, `count`.

`order` 排序操作将应用于查询结果集或者特定合并表的内容排序。排序操作应当立刻跟随`table`对象或者`join`对象。

```swift
public protocol OrderAble: TableProtocol {
	func order(by: PartialKeyPath<Form>...) -> Ordering<OverAllForm, Self>
	func order(descending by: PartialKeyPath<Form>...) -> Ordering<OverAllForm, Self>
}
```

举例：

```swift
let query = try db.table(TestTable1.self)
				.order(by: \.name)
			.join(\.subTables, on: \.id, equals: \.parentId)
				.order(by: \.id)
			.where(\TestTable2.name == .string("Me"))
```

上述例子执行的时候，结果集首先在主清单内排序，随后其子表的数据结果集也进行了排序。

<a name="limit"></a>
### Limit

**Limit** 可跟随： `order`, `join`, `table`.

**Limit** 支持： `join`, `where`, `order`, `select`, `count`.

`limit` 操作可以跟随 `table`、`join` 或者 `order` 操作。限制操作可以用于返回数据集行数，同时可以包括可忽略的数据行总数（用于查询分页）。比如查询结果前5行被忽略，而从第6行开始返回。

```swift
public protocol LimitAble: TableProtocol {
	func limit(_ max: Int, skip: Int) -> Limit<OverAllForm, Self>
}
```

限制数量的操作只能用于上一级链式操作的`table`或者`join`。在`table`后限定的限制数量操作能够控制表格本身返回的数据结果行数。在`join`后限定的数量为合并查询结果集的行数限定。

举例：

```swift
let query = try db.table(TestTable1.self)
				.order(by: \.name)
				.limit(10, skip: 20)
			.join(\.subTables, on: \.id, equals: \.parentId)
				.order(by: \.id)
				.limit(1000)
			.where(\TestTable2.name == .string("Me"))
```

<a name="update"></a>
### Update

**Update** 可跟随： `table`, `where` (when `where` follows `table`).

**Update** 支持：即时操作。

`update` 操作用于替换现有数据记录的具体内容。替换操作总是跟随着 `where` 子句，但是不是必须的。如果不提供 `where` 过滤则更新将应用到数据库表格的所有记录。

```swift
public protocol UpdateAble: TableProtocol {
	func update(_ instance: OverAllForm, setKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Update<OverAllForm, Self>
	func update(_ instance: OverAllForm, ignoreKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Update<OverAllForm, Self>
	func update(_ instance: OverAllForm) throws -> Update<OverAllForm, Self>
}
```

更新操作需要建立在统合类型的具体实例上。该实例用于指定哪些数据符合查询条件。更新操作可以使用一个 `setKeys`参数或者一个`ignoreKeys`参数，也可以不包含任何附加参数用于说明所有数据列都要包含到更新操作中。

举例：

```swift
let newOne = TestTable1(id: 2000, name: "New One", integer: 40)
let newId: Int = try db.transaction {
	try db.table(TestTable1.self).insert(newOne)
	let newOne2 = TestTable1(id: 2000, name: "New One Updated", integer: 41)
	try db.table(TestTable1.self)
		.where(\TestTable1.id == newOne.id)
		.update(newOne2, setKeys: \.name)
	return newOne2.id
}
let j2 = try db.table(TestTable1.self)
	.where(\TestTable1.id == newId)
	.select().map { $0 }
XCTAssertEqual(1, j2.count)
XCTAssertEqual(2000, j2[0].id)
XCTAssertEqual("New One Updated", j2[0].name)
XCTAssertEqual(40, j2[0].integer)
```

<a name="insert"></a>
### Insert

**Insert** 可跟随：`table`.

**Insert** 支持：即时操作

插入操作用于数据库新增数据，每次可以插入一条或者多条数据，可以特别说明指定插入某些字段，或者排除某些字段的内容。插入操作必须跟随表对象 `table1`。

```swift
public extension Table {
	func insert(_ instances: [Form]) throws -> Insert<Form, Table<A,C>>
	func insert(_ instance: Form) throws -> Insert<Form, Table<A,C>>
	func insert(_ instances: [Form], setKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Insert<Form, Table<A,C>>
	func insert(_ instance: Form, setKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Insert<Form, Table<A,C>>
	func insert(_ instances: [Form], ignoreKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Insert<Form, Table<A,C>>
	func insert(_ instance: Form, ignoreKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Insert<Form, Table<A,C>>
}
```

比如：

```swift
let table = db.table(TestTable1.self)
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
let newTwo = TestTable1(id: 2001, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try table.insert([newOne, newTwo], setKeys: \.id, \.name)
```

<a name="delete"></a>
### Delete

**Delete** 可跟随： `table`, `where` (when `where` follows `table`).

**Delete** 支持：即时操作。

删除操作用于删除一条或者多条符合查询条件的数据，通常伴随 `where`子句，虽然不是必须的。如果不跟随where子句进行过滤限制，则所有数据都会被删掉。

```swift
public protocol DeleteAble: TableProtocol {
	func delete() throws -> Delete<OverAllForm, Self>
}
```

举例：

```swift
let table = db.table(TestTable1.self)
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try table.insert(newOne)
let query = table.where(\TestTable1.id == .integer(newOne.id))
let j1 = try query.select().map { $0 }
assert(j1.count == 1)
try query.delete()
let j2 = try query.select().map { $0 }
assert(j2.count == 0)
```

<a name="select"></a>
### Select & Count

**Select** 可跟随： `where`, `order`, `limit`, `join`, `table`.

**Select** 支持：遍历

<a name="select">Select</a> 查询操作返回可遍历结果集。

```swift
public protocol SelectAble: TableProtocol {
	func select() throws -> Select<OverAllForm, Self>
	func count() throws -> Int
	func first() throws -> OverAllForm?
}
```

<a name="count">Count</a> 计数操作和查询操作类似，但是会即刻执行，并仅返回结果集数量，并且不会返回实际的对象实例。

举例：

```swift
let table = db.table(TestTable1.self)
let query = table.where(\TestTable1.blob == .null)
let values = try query.select().map { $0 }
let count = try query.count()
assert(count == values.count)
```

<a name="codable-types"></a>
## Codable Types

大部分可编码类型都可以使用CRUD函数库，通常可以满足需求，不需要修改。所有这些兼容的Swift类型结构体都可以映射到数据库表格的同名字段上。但是您可以将目标映射的字段名修改为不同名称，修改方法是在类型名称上追加“CodingKeys”属性。

默认情况下数据类型名称就会被映射为数据表名，如果希望采取与数据结构不同的名称来命名数据表，则需要实现“TableNameProvider”协议，该协议需要提供一个静态字符串常量 `static let tableName: String` 用于说明表格名称。

CRUD 支持下列Swift 数据类型：

* All Ints, Double, Float, Bool, String
* [UInt8], [Int8], Data
* Date, UUID

上述类型的具体数据库类型映射，则需要根据具体数据库驱动而定。比如在目前的驱动中，Postgres的"date"和"uuid"都映射到了数据库本身的同名类型，但是在SQLite里面保存的类型是字符串。

CRUD兼容类型允许包括一个或者多个子数组，或者联合类型。这些数组能够通过 `join`操作进行统合。注意数据表字段类型不会按照统合类型属性进行创建。

以下范例说明了使用 `Codables` 带 `CodingKeys`声明映射的方法，以及改数据表格名称 `TableNameProvider` 和联合类型的方法：

```swift
struct TestTable1: Codable, TableNameProvider {
	enum CodingKeys: String, CodingKey {
		// specify custom column names for some properties
		case id, name, integer = "int", double = "doub", blob, subTables
	}
	// specify a custom table name
	static let tableName = "test_table_1"
	
	let id: Int
	let name: String?
	let integer: Int?
	let double: Double?
	let blob: [UInt8]?
	let subTables: [TestTable2]?
}

struct TestTable2: Codable {
	let id: UUID
	let parentId: Int
	let date: Date
	let name: String?
	let int: Int?
	let doub: Double?
	let blob: [UInt8]?
}
```

联合类型应该是一组Codable可编码对象的数组。在上面例子里面，测试表1的子表属性是一个联合类型`let subTables: [TestTable2]?`。联合类型只有再对应的表进行联合操作之后，才会返回真实的数据。

<a name="identity"></a>
### Identity


当CRUD 进行表格创建操作时，需要明确主索引。您可以在创建表操作中显式声明主索引字段。如果不声明，则默认表格操作会以 `id`字段为主索引目标。如果没有明确声明主索引，又不存在id字段，则创建表格的操作就不会包括主索引字段。

注意如果以浅表规则（shallow）创建表格时，虽然可以指定主索引，但是却无法递归指定主索引。详见表创建操作。

<a name="error-handling"></a>
## Error Handling

所有在SQL语句创建、执行或者结果提取的时候，如果有错误发生，都会抛出错误对象。

在数据结构编码阶段，CRUD有可能报`CRUDEncoderError`错误，而在数据结构解码阶段，有可能报`CRUDDecodeError`错误。

创建SQL语句时的错误会以`CRUDSQLGenError`形式出现。

执行SQL语句时如果出错则CRUD会抛出 `CRUDSQLExeError` 错误。

所有CRUD出错都会在日志系统中体现。对于数据库驱动的内部错误，是根据驱动自身定义而来。

<a name="logging"></a>
## Logging

CRUD 提供一个内建的日志系统用于记录错误，同时也能够记录每一条SQL语句的创建。CRUD日志记录是异步的，可以通过调用`CRUDLogging.flush()`强制将日志信息写入到磁盘。

您可以随时调用 `CRUDLogging.log(_ type: CRUDLogEventType, _ msg: String)`函数来记录日志事件。

举例

```swift
// 记录一个消息事件
CRUDLogging.log(.info, "我的消息")
```

`CRUDLogEventType` 事件类型包括： `.info`, `.warning`, `.error`, or `.query`.

通过设置静态属性`CRUDLogging.queryLogDestinations`（查询日志目标） 和 `CRUDLogging.errorLogDestinations`（错误日志目标）用于决定日志保存位置。查询日志和出错日志的分开能够便于开发阶段排查错误，而部署到生产服务器时通过忽略查询日志来提高性能。

```swift
public extension CRUDLogging {
	public static var queryLogDestinations: [CRUDLogDestination]
	public static var errorLogDestinations: [CRUDLogDestination]
}
```

日志目标定义为：

```swift
public enum CRUDLogDestination {
	case none
	case console
	case file(String)
	case custom((CRUDLogEvent) -> ())
}
```
每个信息都可以发送到多重日志目标。默认情况下，所有错误和查询都会输出到终端。


