# MySQLStORM

## Relevant Examples

* [MySQLStorm-Demo](https://github.com/PerfectExamples/MySQLStorm-Demo)
* [Perfect-Turnstile-MySQL-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MySQL-Demo)
* [Perfect-Session-MySQL-Demo](https://github.com/PerfectExamples/Perfect-Session-MySQL-Demo)

## Including in your project

When including the dependency in your project's Package.swift dependencies, you will have access to all nested dependencies including the database connector.

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM", majorVersion: 3)
```

### A note about protecting from SQL Injection attacks

StORM does its best to protect your data from SQL injection attacks by parameterizing values.

However in some methods it is possible to specify raw data or SQL query components, so it is important to take care to validate *all* incoming data prior to use in a SQL context.


## Creating a connection to your database

In order to connect to your database you will need to specify certain information.

``` swift
MySQLConnector.host        = "localhost"
MySQLConnector.username    = "username"
MySQLConnector.password    = "secret"
MySQLConnector.database    = "yourdatabase"
MySQLConnector.port        = 3306
```

Once your connection information is specified it can be used in the object class to create the connection on demand.

``` swift
let obj = User()
```

## MySQLStORM supported methods

### Connecting

`MySQLConnector` - Sets the connection parameters for the MySQL server access. Host, port, username and password information, as well as the default database need to be specified.

### Creating tables

`setup()` - Creates the table by inspecting the object. Columns will be created that relate to the assigned type of the property. Properties beginning with an underscore or "internal_" will be ignored.

> **NOTE:** The **primary key** is first property defined in the class.

#### Specifying a custom table creation statement

`setup(String)` - Specify a SQL statement to manually define the table creation process. This will override any StORM setup statement.

### Saving objects

`save()` - Saves object. Creates a new row if no primary key is assigned, otherwise performs an update. 

`save {id in ... }` - Saves object. Creates a new row if no primary key is assigned, otherwise performs an update. The closure returns the new id if created.

`create()` - Forces the creation of a new row, even if a primary key has been supplied.

### Retrieving data

`findAll()` - Retrieves all rows in the table, only limited by the cursor (9,999,999 rows)

`get()` - Retrieves the row. Assumes primary key has been set in the object.

`get(Any)` - Retrieves the row with primary key supplied.

`find([(String, Any)])` - Performs a find, where the pairs supplied are name/value pairs corresponding to column names and find criteria.

`select(whereclause:	String,
		params:			[Any],
		orderby:		[String])` - Performs a select with a specified where clause, having the values parameterized to protect from SQL injection. The orderby array is an array of column names that can optionally include the "ASC" or "DESC" keywords to indicate sort direction.

Additionally the `select` can include:

*  `cursor: StORMCursor` - The optional `cursor` object is a [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md)

*  `columns: [String]` - The optional `columns` is an array of column names to return in the result set. If not included this will default to all columns in the table.

### Deleting objects

`delete()` - Deletes the current object. Assumes an assigned primary key.

`delete(Any)` - Delete the object with primary key supplied.

`delete(Int, idname)` - Delete, specifying a key value, with the name of the key column.

`delete(String, idname)` - Delete, specifying a key value, with the name of the key column.

`delete(UUID, idname)` - Delete, specifying a key value, with the name of the key column.

### Inserts

Each `insert` method will attempt to return the new primary key value.

`insert([(String, Any)])` - Insert a new row specifying data as `[(String, Any)]` where the first in the pair is the column name, and the second is the value.

`insert(cols: [String], params: [Any])` - Insert a new row specifying data as matching arrays of column names, and the associated value.

`insert(cols: [String], params: [Any], idcolumn: String)` - As above, but specifying the id column name to return.


### Updates

`update(data: [(String, Any)], idName: String = "id", idValue: Any)` - Updates the row with the specified data, with a primary key value, and an optional idName column name.

`update(cols: [String], params: [Any], idName: String, idValue: Any)` -  Updates the row by specifying data as matching arrays of column names, and the associated value. A primary key value, and an idName column name are also required.

### Upserts

`upsert(cols: [String], params: [Any], conflictkeys: [String])` - Inserts the row with the specified data; on conflict (conflictkeys columns) it will perform an update.

### SQL Statements

`sql(String, params: [String])` - Execute Raw SQL statement with parameter binding from the params array. Returns raw `PGResult` object.

`sqlRows(String, params: [String])` - Execute Raw SQL statement with parameter binding from the params array. Returns an array of [StORMRow].
