# SQLite

The SQLite connector provides a wrapper around SQLite3, allowing interaction between your Perfect applications and SQLite databases.

## Relevant Examples

* [PerfectArcade](https://github.com/PerfectExamples/PerfectArcade)
* [Perfect-Polling](https://github.com/PerfectExamples/Perfect-Polling)
* [Perfect-Turnstile-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo)
* [Perfect-Session-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Session-SQLite-Demo)


## System Requirements

### macOS

No additional configuration should be required, as SQLite3 is already built in.

### Linux

Make sure that you have SQLite3 installed: `sudo apt-get install sqlite3`

## Setup

Add the "Perfect-SQLite" project as a dependency in your Package.swift file:

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-SQLite.git",
	majorVersion: 3
	)
```

### Import

First and foremost, in any of the source files you intend to use with SQLite, import the module with:

``` swift
import PerfectSQLite
```

### Quick Start

### Access the Database

The database is accessed via its local file path, so the first step is to store the file path to your SQLite data:

``` swift
let dbPath = "./db/database"
```

Once you’ve got that, you can open a connection to the database using a do-try-catch to make sure that errors are handled:

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

### Create Tables

Expanding on our connection above, we’re able to run queries to create database tables by trying the [execute](#execute) method on our connection like so:

``` swift
let dbPath = "./db/database"

do {
	let sqlite = try SQLite(dbPath)
	defer {
		sqlite.close() // This makes sure we close our connection.
	}

	try sqlite.execute(statement: "CREATE TABLE IF NOT EXISTS demo (id INTEGER PRIMARY KEY NOT NULL, option TEXT NOT NULL, value TEXT)")

} catch {
	//Handle Errors
}
```

### Run Queries

* A quick note about string interpolation: Variables in queries do not work as interpolated strings. In order to use variables, you need to use the binding system, described in the [next section](#binding-variables-to-queries).

Once you have a database and tables, the next step is to query and return data. In this example, we will store our statement in a string, and pass it into the the [forEachRow](#foreachrow-with-handlerow) method, which will iterate though each returned row, where you can (most often) append to a dictionary.

``` swift
let dbPath = "./db/database"
var contentDict = [String: Any]()

do {
	let sqlite = try SQLite(dbPath)
		defer {
			sqlite.close() // This makes sure we close our connection.
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
	//Handle Errors
}
```

### Binding Variables to Queries

One thing you’ll definitely want to do is add variables to your queries. As noted above, you cannot do this with string interpolation; instead you can use the binding system. For example:

``` swift
let dbPath = "./db/database"
var contentDict = [String: Any]()

do {
	let sqlite = try SQLite(dbPath)
		defer {
			sqlite.close() // This makes sure we close our connection.
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
	//Handle Errors
}
```

If that looks a little tricky, that’s okay. Our "doBindings:" argument takes a closure to handle adding variables to the positions you’ve defined, and our last argument (technically "handleRow:") is omitted and passed the closure it takes afterward, as it’s the standard way to use Swift. After a few practice runs, it gets much easier to read.

## Full API Reference

The full API consists of:

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

Each of these is detailed below:

### init

``` swift
public init(_ path: String, readOnly: Bool = false) throws
```

Is the basic initializer for creating a new instance of the class. It’s used by passing a file path to an SQLite database and has a secondary parameter used to put the database in read only mode, as necessary. Since readOnly: defaults to false, it’s not necessary to include it if you want to both read and write to the file. This function also throws, which means that you must use it within do-try-catch.

### close

``` swift
public func close()
```

This does exactly what you think it does: it closes the database connection. Its default usage is a defer method inside the do-try-catch that you initialize the database with. This way you are guaranteed to close the connection to the database whether or not your try succeeds or terminates in an error.

### prepare

``` swift
public func prepare(stat: String) throws -> SQLiteStmt
```

This function returns a compiled SQLite statement object that represents the compiled statement. This function is a dependency of other class functions (like execute and forEachRow) which take strings as arguments, pass them to this function, and use the returned object to communicate with the database. There is a very low chance you will ever need to use this function directly, but it is available if you come up with a use case.

### lastInsertRowID

``` swift
public func lastInsertRowID() -> Int
```

This function returns the value of the last row that was inserted into the database. It must be used within your do-try-catch *before* the connection to the database is severed, otherwise it will always return "0". A return value of "0" either means that there is no open connection, or there are no rows. If it is called while another insert is being performed on the same table by a different thread, the result can be slightly unpredictable, so be careful with it. You can visit the [SQLite3 Documentation](https://www.sqlite.org/c3ref/last_insert_rowid.html) for further information about how this function works.

### totalChanges

``` swift
public func totalChanges() -> Int
```

Returns the value of `sqlite3_total_changes`. From [SQLite3’s documentation](https://www.sqlite.org/c3ref/total_changes.html):

*"This function returns the total number of rows inserted, modified or deleted by all INSERT, UPDATE or DELETE statements completed since the database connection was opened, including those executed as part of trigger programs. Executing any other type of SQL statement does not affect the value returned by `sqlite3_total_changes()`."*

Once the connection closes, you are not going to get much value from this, so make sure you include it in the do-try-catch where the related statements are executing.

### changes

``` swift
public func changes() -> Int
```

This function will return the value of `sqlite3_changes`. The major difference between this and totalChanges is that the resulting number of changes will not include anything that the statement triggers; only those rows affected directly by the statement itself.

### errCode

``` swift
public func errCode() -> Int
```

Returns the value of `sqlite3_errcode`. You can learn more about what those mean [here](https://www.sqlite.org/rescode.html).

### errMsg

``` swift
public func errMsg() -> String
```

Returns the value of `sqlite3_errmsg`. Learn more about error codes and messages [here](https://www.sqlite.org/c3ref/errcode.html).

###  execute

``` swift
public func execute(statement: String) throws
```

Runs a statement that expects no return (such as an *INSERT* or *CREATE TABLE*). Since this function throws, make sure to use it within your do-try-catch.

### execute with doBindings:

``` swift
public func execute(statement: String, doBindings: (SQLiteStmt) throws -> ()) throws
```

A variant of execute that also allows for binding variables to the statement before it runs. Also must be in the do-try-catch, as it throws. Learn more about binding variables [here](#binding-variables-to-queries).

### execute with count: and doBindings:

``` swift
public func execute(statement: String, count: Int, doBindings: (SQLiteStmt, Int) throws -> ()) throws
```

This last variant of the execute function allows you to repeat the statement for count: number of times. This is especially useful in situations where you need to loop an insert. Also allows for the [binding of variables](#binding-variables-to-queries). Must be in the do-try-catch, as it throws.

### doWithTransaction

``` swift
public func doWithTransaction(closure: () throws -> ()) throws
```

"Executes a BEGIN, calls the provided closure and executes a ROLLBACK if an exception occurs or a COMMIT if no exception occurs."

That means that you can run a closure with a result, and if that result fails, no changes will actually be made to the database unless that result succeeds. You can read more about SQLite3 transactions [here](https://www.sqlite.org/lang_transaction.html).

### forEachRow with handleRow:

``` swift
public func forEachRow(statement: String, handleRow: (SQLiteStmt, Int) -> ()) throws
```

Executes the given statement, then calls the closure given at handleRow for every row returned. This is especially useful for appending items from the rows returned by the statement into things like dictionaries and arrays. As it throws, it must be placed in your do-try-catch.

### forEachRow with doBindings: and  handleRow:

``` swift
public func forEachRow(statement: String, doBindings: (SQLiteStmt) throws -> (), handleRow: (SQLiteStmt, Int) -> ()) throws
```

As with the previous, this also allows for calling a closure on each row given, but includes the ability to bind variables to the query before it is run. Again, throws means that you need to include this in the do-try-catch.

## Example

The following example is part of a blog system. It’s a function that encapsulates loading page content for a blog post kept in an SQLite database and appending it to a dictionary declared in the class as `var content = [String: Any]()`, which in this particular case would be further used as part of a mustache template that displays that content. It will load content for a page of five posts, given the page number as an argument.

``` swift
func loadPageContent(forPage: Int) {
      do {
          let sqlite = try SQLite(DB_PATH)
          defer {
              sqlite.close()  // defer ensures we close our db connection at the end of this request
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
