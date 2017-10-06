# PostgreSQL

This project provides a Swift wrapper around the libpq client library, enabling access to PostgreSQL servers.

### System Requirements

### macOS

This package requires the [Homebrew](http://brew.sh/) build of PostgreSQL.

To install Homebrew:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

To install Postgres:

```
brew install postgres
```

### Linux

Ensure that you have installed libpq-dev.

```
sudo apt-get install libpq-dev
```

### Setup

Add the "Perfect-PostgreSQL" project as a dependency in your Package.swift file:

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-PostgreSQL.git",
	majorVersion: 3
	)
```

> Remember to rebuild your Xcode project file after making any changes to your Package.swift file.

```
swift package generate-xcodeproj
```

### Import

To use the PostgreSQL connector in a source file, import the module:

``` swift
import PerfectPostgreSQL
```

### Quick Start

### Connect to the database

The connection string defines how you "get" to your database. For more on connection strings and URLs see the PGConnection section below. 

``` swift
do {
	let p = PGConnection()
	let status = p.connectdb("host=localhost dbname=postgres")
	defer {
		p.close() // close the connection
	}
} catch {
	// handle errors
}
```
The status returned from the `p.connectdb()` method is either `.ok` or `.bad`. This allows you to quickly catch and abort your connection processing if the database server is unavailable.

### Running Queries

Once you have your connection established, you normally want to run queries on the database to store or retrieve data.

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

let result = p.exec(
	statement: "
		select datname,datdba,encoding,datistemplate 
		from pg_database
	")

```
The result of the query (`result`) is returned as type `PGResult`. For more on how to work with these result sets, see the `PGResult` documentation below.

### Parameter binding

"Parameter binding" is an alternative method of passing data to the database. Instead of putting the values directly into the SQL statement, you just use a placeholder and provide the actual values in a `params` array.


``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

let result = p.exec(
	statement: "
		insert into test_db (col1, col2, col3)
		values($1, $2, $3)
	",params: [6, "hello world", "another value"])

```
Saving data can be done without Parameter Binding, however, this shifts the burden of sanitizing your data completely to your data validation prior to addressing the database.

### Accessing results

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

let res = p.exec(statement: "
	select datname,datdba,encoding,datistemplate 
	from pg_database 
	where encoding = $1", 
	params: ["6"])
	
let num = res.numTuples()
for x in 0..<num {
	let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
	let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
	let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
	let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
	print("c1=\(c1) c2=\(c2) c3=\(c3) c4=\(c4)")
}
res.clear()
p.close()

```

What's happening:

* `res` is assigned the results of the query, its type is `PGResult`
* `res.numTuples()` provides the number of rows returned in the result
* `PGResult` is iterable, so `for x in 0..<num {}` provides access to each row in turn
* `getFieldString`, `getFieldInt`, `getFieldBool` methods are processing the contents of the row and field number requested


### Managing the Connection: PGConnection

### Opening a Connection

There are two accepted formats for the connection string: plain keyword = value strings, and RFC 3986 URIs.

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

//or
let p = PGConnection()
let status = p.connectdb("postgresql://user:password@localhost:5432/dbname")

```

When using keyword/value connection strings, each parameter is in the form keyword = value. To write an empty value, or a value containing spaces, surround it with single quotes. For instance, keyword = 'a value'.

```
host=localhost port=5432 dbname=mydb connect_timeout=10

```

For a full list of parameter keywords see the relevant PostgreSQL documentation: [https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS)

When specifying the connection string using a connection URI, the general form is:

```
postgresql://[user[:password]@][netloc][:port][/dbname][?param1=value1&...]
```
The URI scheme designator can be either postgresql:// or postgres://. Each of the URI parts is optional. The following examples illustrate valid URI syntax uses:

```
postgresql://
postgresql://localhost
postgresql://localhost:5433
postgresql://localhost/mydb
postgresql://user@localhost
postgresql://user:secret@localhost
postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp
```

### Closing a Connection

Once a connection has been opened, it is important to specify a close of the connection.

This can be done with `.finish()`:

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

defer {
	p.finish()
}

```
> Note that `.close()` is also valid and is functionally equivalent to `.finish()`. It is included for syntax consistency with other connectors.

### Getting the Status of a Connection

After opening a connection you can see its success status:

``` swift
let p = PGConnection()
let connection = p.connectdb("host=localhost dbname=postgres")

print("The connection status is: \(p.status)")

defer {
	p.finish()
}

```

### The Most Recent Error Status

`.errorMessage()` returns the error message most recently generated by an operation on the connection.

``` swift
p.errorMessage()
```

### Executing SQL Statements

Using the `.exec` method, raw SQL statements are passed to the PostgreSQL server. The statements can be provided either as complete strings or parameterized.

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// name, oid, integer, boolean
let result = p.exec(
	statement: "
		select datname,datdba,encoding,datistemplate 
		from pg_database
	")

```
The SQL statements can be anything from SELECT, INSERT, UPDATE, DELETE as well as table creation statements or complete database setup scripts.

"Parameter binding" is an alternative method of passing data to the database. It shifts the burden of sanitizing your data completely to your data validation prior to addressing the database. Instead of putting the values directly into the SQL statement, you just use a placeholder and provide the actual values in a `params` array.


``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// name, oid, integer, boolean
let result = p.exec(
	statement: "
		insert into test_db (col1, col2, col3)
		values($1, $2, $3)
	",params: [6, "hello world", "another value"])

```

### Working with Results: PGResult

PGResult is the container type for all `.exec` method responses.

In the `PGResult` response of a SELECT statement, the number of rows, and the value at a given row and field index can be accessed as below:

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// name, oid, integer, boolean
let res = p.exec(statement: "
	select datname,datdba,encoding,datistemplate 
	from pg_database 
	where encoding = $1", 
	params: ["6"])
	
let num = res.numTuples()
for x in 0..<num {
	let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
	let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
	let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
	let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
	print("c1=\(c1) c2=\(c2) c3=\(c3) c4=\(c4)")
}
res.clear()
p.close()

```

What's happening:

* `res` is assigned the results of the query, its type is `PGResult`
* `res.numTuples()` provides the number of rows returned in the result
* `PGResult` is iterable, so `for x in 0..<num {}` provides access to each row in turn
* `getFieldString`, `getFieldInt`, `getFieldBool` methods are processing the contents of the row and field number requested

### Returning the Status of the Executed Statement

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// name, oid, integer, boolean
let result = p.exec(
	statement: "
		insert into test_db (col1, col2, col3)
		values($1, $2, $3)
	",params: [6, "hello world", "another value"])
	
print("Insert statement result status: \(result.status())")

```
The possible values for `.status()` are:

* `.emptyQuery` - The string sent to the server was empty
* `.commandOK` - Successful completion of a command returning no data
* `.tuplesOK` - Successful completion of a command returning data (such as a SELECT or SHOW)
* `.badResponse` - The server's response was not understood
* `.nonFatalError` - A nonfatal error (a notice or warning) occurred
* `.fatalError` - A fatal error occurred
* `.singleTuple` - The result contains a single result tuple from the current command. This status occurs only when single-row mode has been selected for the query
* `.unknown` - An unknown result status was returned. Look in the console log for more detail

### Getting the Number of Rows Returned

In an earlier example above ("Working with results: PGResult"), the select statement returns the `PGResult` into `res`. The number of rows contained in `res` can be accessed using `.numTuples()`:

``` swift
let num = res.numTuples()
```

### Result Field Count

`.numFields()` returns the number of fields returned. Note that this is not the number of rows, but the fields as in a SELECT statement.

### Determining Field Name and Type

Because fields are not addressed using name but by index, if the precise order of the response is not certain, it may be important to determine the name and type. In this case use `.fieldName(index)` and `.fieldType(index)`:

``` swift
print("The first field name is: \(res.fieldName(0))")
print("The first field type is: \(res.fieldType(0))")
```

### Getting Row Data

Once the result has been returned, we can iterate through the results using a for loop. The `tupleIndex` parameter below is a zero-based index.

Use the `.getField*` methods to access data as appropriate.

``` swift
let num = res.numTuples()
for x in 0..<num {
	let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
	let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
	let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
	let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
	print("c1=\(c1) c2=\(c2) c3=\(c3) c4=\(c4)")
}
```
The `.getField*` methods are:

* `.getFieldString(tupleIndex: Int, fieldIndex: Int)` returns a String type
* `.getFieldInt(tupleIndex: Int, fieldIndex: Int)` returns an Int type
* `.getFieldBool(tupleIndex: Int, fieldIndex: Int)` returns a Boolean type
* `.getFieldInt8(tupleIndex: Int, fieldIndex: Int)` returns an Int8 type
* `.getFieldInt16(tupleIndex: Int, fieldIndex: Int)` returns an Int16 type
* `.getFieldInt32(tupleIndex: Int, fieldIndex: Int)` returns an Int32 type
* `.getFieldInt64(tupleIndex: Int, fieldIndex: Int)` returns an Int64 type
* `.getFieldDouble(tupleIndex: Int, fieldIndex: Int)` returns a Double type
* `.getFieldFloat(tupleIndex: Int, fieldIndex: Int)` returns a Float type
* `.getFieldBlob(tupleIndex: Int, fieldIndex: Int)` returns an [Int8] array

### Testing If a Field Has a Null Value

Similar to the `.getField*` methods, `.fieldIsNull` requires a row (tuple) and field index. The returned value is a Boolean true or false.

``` swift
let nullTest = res.fieldIsNull(tupleIndex: <Int>, fieldIndex: <Int>)
```

### ErrorMessage

To access the error message of the query, use `.errorMessage()`:

``` swift
let result = p.exec(statement: "SELECT * FROM x")
	
print("Error Message: \(result.errorMessage())")

```

### Clearing the Result Cursor

Once a result has been processed, leaving the cursor with all the results in memory can be sub-optimal. Clearing the cursor will release the allocated memory. 

Clear the result using `.clear()`:

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// name, oid, integer, boolean
let result = p.exec(statement: "...")
// process result here

// clearing the cursor:
result.clear()
p.finish()
```
