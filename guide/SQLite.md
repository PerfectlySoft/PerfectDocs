# SQLite

The SQLite connector provides a wrapper around SQLite3, allowing interaction between your Perfect Applications &amp; SQLite databases. 

## System Requirements

### macOS

No additional configuration should be required, as SQlite3 is already built in. 

### Linux 
Make sure that you are running SQLite3:
`sudo apt-get install sqlite3`

## Setup

Add the "Perfect-SQLite" project as a dependency in your Package.swift file:

`.Package(url: "https://github.com/PerfectlySoft/Perfect-SQLite.git", versions: Version(0,0,0)..<Version(10,0,0))`

### Import

Fist and foremost, in any of the source files you intend to use with SQLite, import the module with: 

`import SQLite`

## Quick Start

### Access the Database

The database is accessed via it’s local file path, so the fist step is to store the file path to your sqlite data. 

`let dbPath = "./db/database"`

Once you’ve got that, you can open a connection to the database using a do-try-catch to make sure that errors are handled: 

```
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

Expanding on our connection above, we’re able to run queries to create database tables by trying the *execute* method on our connection like so:

```
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

Once you have a database &amp; tables, the next step is to query and return data. In this example, we will store our statement in a string, and pass it into the the *forEachRow* method, which will iterate though each returned row, where you can (most often) append to a dictionary. 

```
let dbPath = "./db/database"
Var contentDict = [String: Any]()

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

One thing you’ll definitely want to do is add variables to your queries. As noted above, you cannot do this with string interpolation, instead you can use the binding system like so: 

```
let dbPath = "./db/database"
Var contentDict = [String: Any]()

do {
	let sqlite = try SQLite(dbPath)
		defer {
			sqlite.close() // This makes sure we close our connection.
		}
	
	let demoStatement = "SELECT post_title, post_content FROM posts ORDER BY id DESC LIMIT :1"
	
	try sqlite.forEachRow(statement: demoStatement, doBindings {
		
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

If that looks a little tricky, that’s okay. Our "doBindings:" argument takes a closure to handle adding variables to the positions you’ve defined, and our last argument (technically "handleRow:") is omitted and passed the closure it takes afterwards, as it’s the standard Swifty way. After a few practice runs, it gets much easier to read. 