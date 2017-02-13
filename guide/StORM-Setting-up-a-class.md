# Setting up a class to use StORM

The first thing you need to do is import the Library:

``` swift
import PostgresStORM
// or
import SQLiteStORM
```

When you create a new class that will model a database table, it should inherit from a parent StORM class. This will provide most of the goodness you will need.

If you are using PostgreSQL:

``` swift
class User: PostgresStORM {
}
```

If you are using SQLite:

``` swift
class User: SQLiteStORM {
}
```

Next, you will need to add properties to the class. They will mirror closely your columns in the associated table. It's strongly recommended that you keep non-standard characters out of the column names and therefore property names too.

The first property should be your table's primary key. Convention is that this column is called `id` but in reality it can be any valid column name. Common data types for primary keys in SQL datasources are integers, strings, and UUIDs. If your primary key is not an auto-incrementing integer sequence, take care with setting your id and maintaining integrity.

``` swift
// NOTE: First param in class should be the ID.
var id				: Int = 0
var firstname		: String = ""
var lastname		: String = ""
var email			: String = ""
```

You will see above that the default values are set for each property rather than set via an `init()`. This is simply to make this explanation easier. If you choose to write your own `init()` or `init(_ connect: XXConnect)` function, include `super.init()` and beware of the need to add the connection property.

### Specifying the table

The table name is to be manually specified in a function. This is to avoid any possible collision with the introspection that is performed on the class for database operations.

Include a function to specify the table as follows:

``` swift
override open func table() -> String {
	return "users"
}
```

### Class-specific assignment functions

In order for the class to do certain things, such as taking a result set from the database and converting the rows to an array of classed objects, add two functions to the class:

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

Note that in the `to` function, you assign each property. This has the added benefit that at this stage validation or any other special processing can be added.

The `rows` function is simply an array processor. Copy-paste this function and change the 3 occurrences of the class name.


### Putting it all together

The following is an example "complete" PostgresStORM implementation of a class:

``` swift
import PostgresStORM

class User: PostgresStORM {
	// NOTE: First param in class should be the ID.
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