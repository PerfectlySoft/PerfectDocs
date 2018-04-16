# Perfect CRUD

CRUD 是一个基于Swift 4以上版本的关系数据库对象管理系统（ORM）。该函数库采用Swift 4 `Codable` （可编码）协议映射到SQL类型的数据库表格。CRUD能够创建基于`Codable`类型的数据结构并映射到同构数据表格中，实现数据记录插入、更新之类的典型操作。CRUD同样可以进行数据库查询和表格视图合并的操作，所有这些操作都可确保数据类型安全。

CRUD 采用了一种简洁明了但是又表达形式丰富多样、数据类型在编译阶段得到安全检查保障的方法来构造查询操作，其实现目标是轻量化、免依存关系。实现方法采用了类型模板（Generics）、字段路径（KeyPath）和可编码（Codable）以确保编译阶段对上述内容的一致检查。

目前可以使用的实际数据实现可以在这里找到：[SQLite](https://github.com/PerfectlySoft/Perfect-SQLite)、[Postgres](https://github.com/PerfectlySoft/Perfect-PostgreSQL)和[MySQL](https://github.com/PerfectlySoft/Perfect-MySQL)。

## 使用方法

以下是CRUD的简明实用方法

```swift
// CRUD 可以应用到大多数可编码类型：
struct PhoneNumber: Codable {
	let id: UUID
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
// 首先创建一个 `Database` 对象并进行配置，以下示范代码采用了SQLite作为演示：

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
do {
	// 示范如何插入数据
	let personId1 = UUID()
	let personId2 = UUID()
	try personTable.insert([
		Person(id: personId1, firstName: "Owen", lastName: "Lars", phoneNumbers: nil),
		Person(id: personId2, firstName: "Beru", lastName: "Lars", phoneNumbers: nil)])
	try numbersTable.insert([
		PhoneNumber(id: UUID(), personId: personId1, planetCode: 12, number: "555-555-1212"),
		PhoneNumber(id: UUID(), personId: personId1, planetCode: 15, number: "555-555-2222"),
		PhoneNumber(id: UUID(), personId: personId2, planetCode: 12, number: "555-555-1212")
	])
}

// 查询示范：检索所有姓氏为Lars的人，同时电话号码为区号12
let query = try personTable
		.order(by: \.lastName)
	.join(\.phoneNumbers, on: \.id, equals: \.personId)
		.order(descending: \.planetCode)
	.where(\Person.lastName == .string("Lars") && \PhoneNumber.planetCode == .integer(12))
	.select()
	
// 遍历查询结果并打印姓名
for user in query {
	print("\(user.firstName) \(user.lastName)")
	// 因为合并了电话本这个表格，所以此处可以一并列出
	guard let numbers = user.phoneNumbers else {
		continue
	}
	for number in numbers {
		print(number.number)
	}
}
```

## 操作方法

CRUD 的主要操作集中于数据库对象连接之后进行一系列的数据库操作。某些操作是立刻执行的，而另外一些操作（比如select查询）是根据需要才实际执行。每个操作的结果都是可以继续作为参考进行下一步操作，即链式操作。

以下操作介绍的分类是依据实现相应操作的有关对象。注意为了便于示范，很多类型定义都是缩写，而且部分函数以扩展方式实现，以确保整体性集中。

### Database （数据对象）

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

#### transaction （事务对象）

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

#### create （表格创建）

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

`TableCreatePolicy` 表格创建规则包括下列选项：

* .shallow （浅表创建）- 如果声明，则合并类型的表格不会自动创建；如果不声明，则所有合并类型的表格都会自动创建，并使用“id“字段作为表格主关键字索引。如果主索引无法应用则会出错。
* .dropTable （先删后建）- 创建表格前先删除同名表格。这个方式在软件开发和测试阶段特别好用，或者对于某些只保存临时数据的表格也很适用，只要程序重启则旧表格会被自动覆盖。
* .reconcileTable（求同除异）- 如果数据库存在同名表格，则对于同名表格内名称相同但类型不同的字段，会被调整为与新类型；如果同名表格内缺失新表格字段，则新字段被追加；如果同名旧表格存在不同名称的字段，则会被删除。

只要不应用.dropTable和.reconcileTable规则，即使表格存在，调用表格创建操作实际上是安全的，现存表格结构和记录都不会修改。

#### table （数据表对象）

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

### Table（数据表对象）

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

**Table** 从 `Database` 中创建实例

**Table** 支持的操作： `update`, `insert`, `delete`, `join`, `order`, `limit`, `where`, `select`, and `count`.

### Join （横向合并）

`join` 表格横向合并操作能够将不同表对象的查询结果集合进行整体合并，生成统合类型对象。合并结果是一个父一级统合类型的一个属性。注意横向合并只有在真正执行“select”查询操作时才会真的执行。合并后的操作目前暂时不支持更新、插入和删除（也不支持递归删除和递归更新）。

```swift
public protocol JoinAble: TableProtocol {
	func join<NewType: Codable, KeyType: Equatable>(
		_ to: KeyPath<OverAllForm, [NewType]?>,
		on: KeyPath<OverAllForm, KeyType>,
		equals: KeyPath<NewType, KeyType>) throws -> Join<OverAllForm, Self, NewType, KeyType>
}
```

横向合并需要三个参数：

`to` - 指向统合类型属性的字段路径。该字段路径指向的应该是一个可选类型数组，其元素类型应该是非集成的可编码数据结构。该属性用于设置结果集对象。

`on` - 用于统合类型的主索引的字段路径（通常就是主数据表的主索引字段）。

`equals` - 等价于统合类型的“on“字段路径，用于关系数据库的外部索引。

举例：

```swift
let table = db.table(TestTable1.self)
let join = try table.join(\.subTables, on: \.id, equals: \.parentId)
```

上述例子中，测试表2与测试表1的.subTables子表进行了横向合并，类型是 `[TestTable2]?`。当该查询真正执行时，所有测试表2中parentId与测试表1主索引字段id一致的数据记录将被包含在查询结果集内。

如果不明确横向合并的结果类型，则合并后的表类型对于任何统合对象的实例都会置为nil。

如果表格完成了合并但是结果集内没有数据记录，则统合类型属性为一个空数组。

**Join** 可跟随操作: `table`, `order`, `limit`, 或者是另外一个 `join`.

**Join** 支持操作： `join`, `where`, `order`, `limit`, `select`, `count`.

### Where （过滤器对象）

`where` 操作用于定义数据结果的过滤条件，过滤数据的结果可以用于查询、更新或者删除。只有在进行查询/统计、更新或删除操作时，才可以使用Where子句（过滤器对象）。

```swift
public protocol WhereAble: TableProtocol {
	func `where`(_ expr: Expression) -> Where<OverAllForm, Self>
}
```

Where 操作虽然是可选的，但是每个链式操作只允许有一个 `where` 子句，并且必须置于整个链式操作的末端作为限制条件。

举例：

```swift
let table = db.table(TestTable1.self)
// 插入一条记录，然后查询这条数据
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try table.insert(newOne)

// 根据 id 检索数据对象
let query = table.where(\TestTable1.id == .integer(newOne.id))
guard let foundNewOne = try query.select().first else {
	...
}
```

`where` 操作的参数是一个 `Expression`（表达式）对象。表达式对象是用于定义所有合法操作的一个枚举类型，这种方式使得Swift的表达式语法可以直接映射为SQL表达式。CRUD函数库用这种方式实现SQL语句的参数化和实例生成，所以最终用户不用去考虑如何绑定数据，比如加引号或者对二进制数据进行编码之类。

多数表达式类型结果都是简单集合类型，比如`.string(String)` 或者 `.null`；其他表达式为二进制或者逻辑操作，比如`AND`或者 `==`。对应的Swift 表达式是 `&&` and `==`。

为了说明具体使用方法，参考下列两行等价语句：

```swift
let query1 = table.where(\TestTable1.id == .integer(newOne.id))
let query2 = table.where(
	.equality(.keyPath(\TestTable1.id), .integer(newOne.id)))
```

第一个查询使用了Swift语法创建where子句，而第二个方法看起来更加啰嗦一些，但是结果是一样的。

CRUD `Expression` 提供如下重载运算表达式：

```swift
public extension Expression {
	static func &&(lhs: Expression, rhs: Expression) -> Expression
	static func ||(lhs: Expression, rhs: Expression) -> Expression
	static func ==(lhs: Expression, rhs: Expression) -> Expression
	static func !=(lhs: Expression, rhs: Expression) -> Expression
	static func <(lhs: Expression, rhs: Expression) -> Expression
	static func <=(lhs: Expression, rhs: Expression) -> Expression
	static func >(lhs: Expression, rhs: Expression) -> Expression
	static func >=(lhs: Expression, rhs: Expression) -> Expression
	static prefix func !(rhs: Expression) -> Expression
}
```

此处不在赘述上述表达式的其他重载版本，比如左子式运算或者右子式运算等等，用于生成字符串、整数或者浮点数之类的简洁表达。

其他有关的字段路径左子式运算表达包括：

```swift
public extension Expression {
	static func == <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
	static func != <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
	static func > <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
	static func >= <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
	static func < <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
	static func <= <T: Codable>(lhs: PartialKeyPath<T>, rhs: Expression) -> Expression
}
```

**Where** 可以跟随：`table`, `join`, `order`.

**Where** 支持 `select`, `count`, `update` (跟随 `table`), `delete` (跟随 `table`).

### Order （排序对象）

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

**Order** 可跟随： `table`, `join`.

**Order** 可支持： `join`, `where`, `order`, `limit` `select`, `count`.

### Limit（限制数量）

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

**Limit** 可跟随怒： `order`, `join`, `table`.

**Limit** 支持： `join`, `where`, `order`, `select`, `count`.

### Update （更新对象）

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
let table = db.table(TestTable1.self)
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try db.transaction {
	try table.insert(newOne)
	let newOne2 = TestTable1(id: 2000, name: "New One Updated", integer: 41, double: nil, blob: nil, subTables: nil)
	try table
		.where(\TestTable1.id == .integer(newOne.id))
		.update(newOne2, setKeys: \.name)
}
let results = try table
	.where(\TestTable1.id == .integer(newOne.id))
	.select().map { $0 }

assert(results.count == 1)
assert(results[0].id == 2000)
assert(results[0].name == "New One Updated")
assert(results[0].integer == 40)
```

**Update** 可跟随： `table`, `where` (当 `where` 子句跟随了 `table`对象).

**Update** 支持立刻查询。

### Insert （插入对象）

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

**Insert** 可跟随： `table`.

**Insert** 支持立刻执行。

### Delete （删除对象）

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

**Delete** 可跟随： `table`, `where` (当 `where` 子句跟随 `table`).

**Delete** 支持立刻执行

### Select & Count （查询和计数）

查询操作返回可遍历结果集。

```swift
public protocol SelectAble: TableProtocol {
	func select() throws -> Select<OverAllForm, Self>
	func count() throws -> Int
}
```

计数操作和查询操作类似，但是会即刻执行，并仅返回结果集数量，并且不会返回实际的对象实例。

举例：

```swift
let table = db.table(TestTable1.self)
let query = table.where(\TestTable1.blob == .null)
let values = try query.select().map { $0 }
let count = try query.count()
assert(count == values.count)
```

**Select** 可跟随： `where`, `order`, `limit`, `join`, `table`.

**Select** 支持遍历

## Codable Types （可编码类型）

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

### Identity （序号字段）

所有CRUD可编码类型都强制包含一个 `id` 字段。当CRUD 进行表格创建操作时，需要明确主索引。您可以在创建表操作中显式声明主索引字段。如果不声明，则默认表格操作会以 `id`字段为主索引目标。如果没有明确声明主索引，又不存在id字段，则创建表格的操作会报错。

注意如果以浅表规则（shallow）创建表格时，虽然可以指定主索引，但是却无法递归指定主索引。详见表创建操作。

## 错误处理

所有在SQL语句创建、执行或者结果提取的时候，如果有错误发生，都会抛出错误对象。

在数据结构编码阶段，CRUD有可能报`CRUDEncoderError`错误，而在数据结构解码阶段，有可能报`CRUDDecodeError`错误。

创建SQL语句时的错误会以`CRUDSQLGenError`形式出现。

执行SQL语句时如果出错则CRUD会抛出 `CRUDSQLExeError` 错误。

所有CRUD出错都会在日志系统中体现。对于数据库驱动的内部错误，是根据驱动自身定义而来。

## 日志

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


