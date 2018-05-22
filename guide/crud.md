# Perfect CRUD

CRUD is an object-relational mapping (ORM) system for Swift 4+. CRUD takes Swift 4 `Codable` types and maps them to SQL database tables. CRUD can create tables based on `Codable` types and perform inserts and updates of objects in those tables. CRUD can also perform selects and joins of tables, all in a type-safe manner.

CRUD uses a simple, expressive, and type safe methodology for constructing queries as a series of operations. It is designed to be light-weight and has zero additional dependencies. It uses generics, KeyPaths and Codables to ensure as much misuse as possible is caught at compile time.

Database client library packages can add CRUD support by implementing a few protocols. Support is available for [SQLite](https://github.com/PerfectlySoft/Perfect-SQLite), [Postgres](https://github.com/PerfectlySoft/Perfect-PostgreSQL), and [MySQL](https://github.com/PerfectlySoft/Perfect-MySQL).

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

This is a simple example to show how CRUD is used.

```swift
// CRUD can work with most Codable types.
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

// CRUD usage begins by creating a database connection.
// The inputs for connecting to a database will differ depending on your client library.
// Create a `Database` object by providing a configuration.
// These examples will use SQLite for demonstration purposes,
// 	but all code would be identical regardless of the datasource type.
let db = Database(configuration: try SQLiteDatabaseConfiguration(testDBName))

// Create the table if it hasn't been done already.
// Table creates are recursive by default, so "PhoneNumber" is also created here.
try db.create(Person.self, policy: .reconcileTable)

// Get a reference to the tables we will be inserting data into.
let personTable = db.table(Person.self)
let numbersTable = db.table(PhoneNumber.self)

// Add an index for personId, if it does not already exist.
try numbersTable.index(\.personId)

// Insert some sample data.
do {
	// Insert some sample data.
	let owen = Person(id: UUID(), firstName: "Owen", lastName: "Lars", phoneNumbers: nil)
	let beru = Person(id: UUID(), firstName: "Beru", lastName: "Lars", phoneNumbers: nil)
	
	// Insert the people
	try personTable.insert([owen, beru])
	
	// Give them some phone numbers
	try numbersTable.insert([
		PhoneNumber(personId: owen.id, planetCode: 12, number: "555-555-1212"),
		PhoneNumber(personId: owen.id, planetCode: 15, number: "555-555-2222"),
		PhoneNumber(personId: beru.id, planetCode: 12, number: "555-555-1212")])
}

// Perform a query.
// Let's find all people with the last name of Lars which have a phone number on planet 12.
let query = try personTable
		.order(by: \.lastName, \.firstName)
	.join(\.phoneNumbers, on: \.id, equals: \.personId)
		.order(descending: \.planetCode)
	.where(\Person.lastName == "Lars" && \PhoneNumber.planetCode == 12)
	.select()

// Loop through the results and print the names.
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

Activity in CRUD is accomplished by obtaining a database connection object and then chaining a series of operations on that database. Some operations execute immediately while others (select) are executed lazily. Each operation that is chained will return an object which can be further chained or executed.

Operations are grouped here according to the objects which implement them. Note that many of the type definitions shown below have been abbreviated for simplicity and some functions implemented in extensions have been moved in to keep things in a single block.

<a name="database"></a>
### Database

A Database object wraps and maintains a connection to a database. Database connectivity is specified by using a `DatabaseConfigurationProtocol` object. These will be specific to the database in question.

```swift
// postgres sample configuration
let db = Database(configuration: 
	try PostgresDatabaseConfiguration(database: postgresTestDBName, host: "localhost"))
// sqlite sample configuration
let db = Database(configuration: 
	try SQLiteDatabaseConfiguration(testDBName))
```

Database objects implement this set of logical functions:

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

The operations available on a Database object include `transaction`, `create`, and `table`. 

<a name="transaction"></a>
#### Transaction

The `transaction` operation will execute the body between a set of "BEGIN" and "COMMIT" or "ROLLBACK" statements. If the body completes execution without throwing an error then the transaction will be committed, otherwise it is rolled-back.

```swift
public extension Database {
	func transaction<T>(_ body: () throws -> T) throws -> T
}
```

Example usage:

```swift
try db.transaction {
	... further operations
}
```

The body of a transaction may optionally return a value.

```swift
let value = try db.transaction {
	... further operations
	return 42
}
```

<a name="create"></a>
#### Create

The `create` operation is given a Codable type. It will create a table corresponding to the type's structure. The table's primary key can be indicated as well as a "create policy" which determines some aspects of the operation. 

```swift
public extension DatabaseProtocol {
	func create<A: Codable>(
		_ type: A.Type, 
		primaryKey: PartialKeyPath<A>? = nil, 
		policy: TableCreatePolicy = .defaultPolicy) throws -> Create<A, Self>
}
```

Example usage:

```swift
try db.create(TestTable1.self, primaryKey: \.id, policy: .reconcileTable)
```

<a name="create-policy">`TableCreatePolicy`</a> consists of the following options:

* .reconcileTable - If the database table already exists then any columns which differ between the type and the table will be either removed or added. Note that this policy will not alter columns which exist but have changed their types. For example, if a type's property changes from a String to an Int, this policy will not alter the column changing its type to Int.
* .shallow - If indicated, then joined type tables will not be automatically created. If not indicated, then any joined type tables will be automatically created.
* .dropTable - The database table will be dropped before it is created. This can be useful during development and testing, or for tables that contain ephemeral data which can be reset after a restart.

Calling create on a table which already exists is a harmless operation resulting in no changes unless the `.reconcileTable` or `.dropTable` policies are indicated. Existing tables will not be modified to match changes in the corresponding Codable type unless `.reconcileTable` is indicated.

<a name="table"></a>
#### Table

The `table` operation returns a Table object based on the indicated Codable type. Table objects are used to perform further operations.

```swift
public protocol DatabaseProtocol {
	func table<T: Codable>(_ form: T.Type) -> Table<T, Self>
}
```

Example usage:

```swift
let table1 = db.table(TestTable1.self)
```

<a name="sql"></a>
#### SQL

CRUD can also execute bespoke SQL statements, mapping the results to an array of any suitable Codable type.

```swift
public extension Database {
	func sql(_ sql: String, bindings: Bindings = []) throws
	func sql<A: Codable>(_ sql: String, bindings: Bindings = [], _ type: A.Type) throws -> [A]
}
```

Example Usage:

```swift
try db.sql("SELECT * FROM mytable WHERE id = 2", TestTable1.self)
```

<a name="table-1"></a>
### Table

**Table** can follow: `Database`.

**Table** supports: `update`, `insert`, `delete`, `join`, `order`, `limit`, `where`, `select`, and `count`.

A Table object can be used to perform updates, inserts, deletes or selects. Tables can only be accessed through a database object by providing the Codable type which is to be mapped. A table object can only appear in an operation chain once, and it must be the first item.

Table objects are parameterized based on the Swift object type you provide when you retrieve the table. Tables indicate the over-all resulting type of any operation. This will be referred to as the *OverAllForm*.

Example usage:

```swift
// get a table object representing the TestTable1 struct
// any inserts, updates, or deletes will affect "TestTable1"
// any selects will produce a collection of TestTable1 objects.
let table1 = db.table(TestTable1.self)
```

In the example above, TestTable1 is the OverAllForm. Any destructive operations will affect the corresponding database table. Any selects will produce a collection of TestTable1 objects.

<a name="join"></a>
### Join

**Join** can follow: `table`, `order`, `limit`, or another `join`.

**Join** supports: `join`, `where`, `order`, `limit`, `select`, `count`.

A `join` brings in objects from another table in either a parent-child or many-to-many relationship scheme.

<a name="parent-child">Parent-child</a> example usage:

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

The example above joins Child objects on the Parent.children property, which is of type `[Child]?`. When the query is executed, all objects from the Child table that have a parentId which matches the Parent id 1 will be included in the results. This is a typical parent-child relationship.

<a name="many-to-many">Many-to-many</a> example usage:

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

<a name="self-join">Self Join</a> example usage:

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

<a name="junction-join">Junction Join</a> example usage:

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

Joins are not currently supported in updates, inserts, or deletes (cascade deletes/recursive updates are not supported).

The Join protocol has two functions. The first handles standard two table joins. The second handles junction (three table) joins.

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

A standard join requires three parameters: 

`to` - keypath to a property of the OverAllForm. This keypath should point to an Optional array of non-integral Codable types. This property will be set with the resulting objects.

`on` - keypath to a property of the OverAllForm which should be used as the primary key for the join (typically one would use the actual table primary key column).

`equals` - keypath to a property of the joined type which should be equal to the OverAllForm's `on` property. This would be the foreign key.

A junction join requires six parameters: 

`to` - keypath to a property of the OverAllForm. This keypath should point to an Optional array of non-integral Codable types. This property will be set with the resulting objects.

`with` - The type of the junction table.

`on` - keypath to a property of the OverAllForm which should be used as the primary key for the join (typically one would use the actual table primary key column).

`equals` - keypath to a property of the junction type which should be equal to the OverAllForm's `on` property. This would be the foreign key.

`and` - keypath to a property of the child table type which should be used as the key for the join (typically one would use the actual table primary key column).

`is` - keypath to a property of the junction type which should be equal to the child table's `and` property.

Any joined type tables which are not explicitly included in a join will be set to nil for any resulting OverAllForm objects.

If a joined table is included in a join but there are no resulting joined objects, the OverAllForm's property will be set to an empty array.

<a name="where"></a>
### Where

**Where** can follow: `table`, `join`, `order`.

**Where** supports: `select`, `count`, `update` (when following `table`), `delete` (when following `table`).

A `where` operation introduces a criteria which will be used to filter exactly which objects should be selected, updated, or deleted from the database. Where can only be used when performing a select/count, update, or delete. 

```swift
public protocol WhereAble: TableProtocol {
	func `where`(_ expr: CRUDBooleanExpression) -> Where<OverAllForm, Self>
}
```

`Where` operations are optional, but only one `where` can be included in an operation chain and it must be the penultimate operation in the chain.

Example usage:

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
The parameter given to the `where` operation is a `CRUDBooleanExpression` object. These are produced by using any of the supported expression operators.

Standard Swift Operators:

• Equality: `==`, `!=`

• Comparison: `<`, `<=`, `>`, `>=`

• Logical: `!`, `&&`, `||`

Custom Comparison Operators:

• Contained/In: `~`, `!~`

• Like: `%=%`, `=%`, `%=`, `%!=%`, `!=%`, `%!=` 

For the equality and comparison operators, the left-hand operand must be a KeyPath indicating a Codable property of a Codable type. The right-hand operand can be Int, Double, String, [UInt8], Bool, UUID, or Date. The KeyPath can indicate an Optional property value, in which case the right-hand operand may be `nil` to indicate an "IS NULL", "IS NOT NULL" type of query.

The equality and comparison operators are type-safe, meaning you can not make a comparison between, for example, an Int and a String. The type of the right-hand operand must match the KeyPath property type. This is how Swift normally works, so it should not come with any surprises.

Any type which has been introduced to the query through a `table` or `join` operation can be used in an expression. Using KeyPaths for types not used elsewhere in the query is a runtime error. 

In this snippet:

```swift
table.where(\TestTable1.id > 20)
```

`\TestTable1.id` is the KeyPath, pointing to the Int id for the object. 20 is the literal operand value. The `>` operator between them produces a `CRUDBooleanExpression` which can be given directly to `where` or used with other operators to make more complex expressions.

The logical operators permit `and`, `or`, and `not` operations given two `CRUDBooleanExpression` objects. These use the standard Swift `&&`, `||`, and `!` operators.

```swift
table.where(\TestTable1.id > 20 && 
	!(\TestTable1.name == "Me" || \TestTable1.name == "You"))
```

The contained/in operators take a KeyPath in the right-hand side and an array of objects on the left. 

```swift
table.where(\TestTable1.id ~ [2, 4])
table.where(\TestTable1.id !~ [2, 4])
```

The above will select all TestTable1 objects whose `id` is in the array, or not in the array, respectively.

The Like operators are used only with String values. These permit begins with, ends with, and contains searches on String based columns.

```swift
try table.where(\TestTable2.name %=% "me") // contains
try table.where(\TestTable2.name =% "me") // begins with
try table.where(\TestTable2.name %= "me") // ends with
try table.where(\TestTable2.name %!=% "me") // not contains
try table.where(\TestTable2.name !=% "me") // not begins with
try table.where(\TestTable2.name %!= "me") // not ends with
```

**Property Optionals**

In some cases you may need to query using an optional parameter in your model. In the `Person` model above, we may need to add an optional `height` field:

```swift
struct Person: Codable {
	// ...
	var height: Double? // height in cm
}
```

In the case we wanted to search for `People` who have not yet provided their height, this simple query will find all rows where `height` is `NULL`.

```swift
let people = try personTable.where(\Person.height == nil).select().map{ $0 }
```

Alternatively, you may need to query for individuals a certain height or taller.

```swift
let queryHeight = 170.0
let people = try personTable.where(\Person.height! >= queryHeight).select().map{ $0 }
```

Notice the force-unwraped key path - `\Person.height!`. _This is type-safe_ and required by the compiler in order to compare the optional type on the model to the non-optional value in the query.

<a name="order"></a>
### Order

**Order** can follow: `table`, `join`.

**Order** supports: `join`, `where`, `order`, `limit` `select`, `count`.

An `order` operation introduces an ordering of the over-all resulting objects and/or of the objects selected for a particular join. An order operation should immediately follow either a `table` or a `join`. You may also order over fields with optional types.

```swift
public protocol OrderAble: TableProtocol {
	func order(by: PartialKeyPath<Form>...) -> Ordering<OverAllForm, Self>
	func order(descending by: PartialKeyPath<Form>...) -> Ordering<OverAllForm, Self>
}
```

Example usage:

```swift
let query = try db.table(TestTable1.self)
				.order(by: \.name)
			.join(\.subTables, on: \.id, equals: \.parentId)
				.order(by: \.id)
			.where(\TestTable2.name == "Me")
```

When the above query is executed it will apply orderings to both the main list of returned objects and to their individual "subTables" collections.

Ordering by a nullable field:

```swift
struct Person: Codable {
	// ...
	var height: Double? // height in cm
}

// ...

let person = try personTable.order(descending: \.height).select().map {$0}
```

<a name="limit"></a>
### Limit

**Limit** can follow: `order`, `join`, `table`.

**Limit** supports: `join`, `where`, `order`, `select`, `count`.

A `limit` operation can follow a `table`, `join`, or `order` operation. Limit can both apply an upper bound on the number of resulting objects and impose a skip value. For example the first five found records may be skipped and the result set will begin at the sixth row.

```swift
public protocol LimitAble: TableProtocol {
	func limit(_ max: Int, skip: Int) -> Limit<OverAllForm, Self>
}
```

A limit applies only to the most recent `table` or `join`. A limit placed after a `table` limits the over-all number of results. A limit placed after a `join` limits the number of joined type objects returned.

Example usage:

```swift
let query = try db.table(TestTable1.self)
				.order(by: \.name)
				.limit(10, skip: 20)
			.join(\.subTables, on: \.id, equals: \.parentId)
				.order(by: \.id)
				.limit(1000)
			.where(\TestTable2.name == "Me")
```

<a name="update"></a>
### Update

**Update** can follow: `table`, `where` (when `where` follows `table`).

**Update** supports: immediate execution.

An `update` operation can be used to replace values in the existing records which match the query. An update will almost always have a `where` operation in the chain, but it is not required. Providing no `where` operation in the chain will match all records. 

```swift
public protocol UpdateAble: TableProtocol {
	func update(_ instance: OverAllForm, setKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Update<OverAllForm, Self>
	func update(_ instance: OverAllForm, ignoreKeys: PartialKeyPath<OverAllForm>, _ rest: PartialKeyPath<OverAllForm>...) throws -> Update<OverAllForm, Self>
	func update(_ instance: OverAllForm) throws -> Update<OverAllForm, Self>
}
```

An update requires an instance of the OverAllForm. This instance provides the values which will be set in any records which match the query. The update can be performed with either a `setKeys` or `ignoreKeys` parameter, or with no additional parameter to indicate that all columns should be included in the update.

Example usage:

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

**Insert** can follow: `table`.

**Insert** supports: immediate execution.

Insert is used to add new records to the database. One or more objects can be inserted at a time. Particular keys/columns can be added or excluded. An insert must immediately follow a `table`.

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

Usage example:

```swift
let table = db.table(TestTable1.self)
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
let newTwo = TestTable1(id: 2001, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try table.insert([newOne, newTwo], setKeys: \.id, \.name)
```

<a name="delete"></a>
### Delete

**Delete** can follow: `table`, `where` (when `where` follows `table`).

**Delete** supports: immediate execution.

A `delete` operation is used to remove records from the table which match the query. A delete will almost always have a `where` operation in the chain, but it is not required. Providing no `where` operation in the chain will delete all records.

```swift
public protocol DeleteAble: TableProtocol {
	func delete() throws -> Delete<OverAllForm, Self>
}
```

Example usage:

```swift
let table = db.table(TestTable1.self)
let newOne = TestTable1(id: 2000, name: "New One", integer: 40, double: nil, blob: nil, subTables: nil)
try table.insert(newOne)
let query = table.where(\TestTable1.id == newOne.id)
let j1 = try query.select().map { $0 }
assert(j1.count == 1)
try query.delete()
let j2 = try query.select().map { $0 }
assert(j2.count == 0)
```

<a name="select"></a>
### Select & Count

**Select** can follow: `where`, `order`, `limit`, `join`, `table`.

**Select** supports: iteration.

<a name="select">Select</a> returns an object which can be used to iterate over the resulting values.

```swift
public protocol SelectAble: TableProtocol {
	func select() throws -> Select<OverAllForm, Self>
	func count() throws -> Int
	func first() throws -> OverAllForm?
}
```

<a name="count">Count</a> works similarly to `select` but it will execute the query immediately and simply return the number of resulting objects. Object data is not actually fetched.

Usage example:

```swift
let table = db.table(TestTable1.self)
let query = table.where(\TestTable1.blob == nil)
let values = try query.select().map { $0 }
let count = try query.count()
assert(count == values.count)
```

<a name="codable-types"></a>
## Codable Types

Most `Codable` types can be used with CRUD, often, depending on your needs, with no modifications. All of a type's relevant properties will be mapped to columns in the database table. You can customize the column names by adding a `CodingKeys` property to your type. 

By default, the type name will be used as the table name. To customize the name used for a type's table, have the type implement the `TableNameProvider` protocol. This requires a `static let tableName: String` property.

CRUD supports the following property types:

* All Ints, Double, Float, Bool, String
* [UInt8], [Int8], Data
* Date, UUID

The actual storage in the database for each of these types will depend on the client library in use. For example, Postgres will have an actual "date" and "uuid" column types while in SQLite these will be stored as strings.

A type used with CRUD can also have one or more arrays of child, or joined types. These arrays can be populated by using a `join` operation in a query. Note that a table column will not be created for joined type properties.

The following example types illustrate valid CRUD `Codables` using `CodingKeys`, `TableNameProvider` and joined types.

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

Joined types should be an Optional array of Codable objects. Above, the `TestTable1` struct has a joined type on its `subTables` property: `let subTables: [TestTable2]?`. Joined types will only be populated when the corresponding table is joined using the `join` operation.

<a name="identity"></a>
### Identity

When CRUD creates the table corresponding to a type it attempts to determine what the primary key for the table will be. You can explicitly indicate which property is the primary key when you call the `create` operation. If you do not indicate the key then a property named "id" will be sought. If there is no "id" property then the table will be created without a primary key.
Note that a custom primary key name can be specified when creating tables "shallow" but not when recursively creating them. See the "Create" operation for more details.

<a name="error-handling"></a>
## Error Handling

Any error which occurs during SQL generation, execution, or results fetching will produce a thrown Error object. 

CRUD will throw `CRUDDecoderError` or `CRUDEncoderError` for errors occurring during type encoding and decoding, respectively.

CRUD will throw `CRUDSQLGenError` for errors occurring during SQL statement generation.

CRUD will throw `CRUDSQLExeError` for errors occurring during SQL statement execution.

All of the CRUD errors are tied into the logging system. When they are thrown the error messages will appear in the log. Individual database client libraries may throw other errors when they occur.

<a name="logging"></a>
## Logging

CRUD contains a built-in logging system which is designed to record errors which occur. It can also record individual SQL statements which are generated. CRUD logging is done asynchronously. You can flush all pending log messages by calling `CRUDLogging.flush()`.

Messages can be added to the log by calling `CRUDLogging.log(_ type: CRUDLogEventType, _ msg: String)`.

Example usage:

```swift
// log an informative message.
CRUDLogging.log(.info, "This is my message.")
```

`CRUDLogEventType` is one of: `.info`, `.warning`, `.error`, or `.query`.

You can control where log messages go by setting the `CRUDLogging.queryLogDestinations` and `CRUDLogging.errorLogDestinations` static properties. Modifying the log destinations is a thread-safe operation. Handling for errors and queries can be set separately as SQL statement logging may be desirable during development but not in production.

```swift
public extension CRUDLogging {
	public static var queryLogDestinations: [CRUDLogDestination]
	public static var errorLogDestinations: [CRUDLogDestination]
}
```

Log destinations are defined as:

```swift
public enum CRUDLogDestination {
	case none
	case console
	case file(String)
	case custom((CRUDLogEvent) -> ())
}
```

Each message can go to multiple destinations. By default, both errors and queries are logged to the console.
