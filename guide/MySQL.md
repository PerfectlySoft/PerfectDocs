# MySQL

The MySQL connector provides a wrapper around MySQL, allowing interaction between your Perfect Applications &amp; MySQL databases. 

## System Requirements

### macOS

Requires the use of Homebrew’s MySQL. 

```shell
brew install mysql
```

If you need home-brew, you can install it with: 

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Unfortunately, at this point in time you will need to edit the mysqlclient.pc file located here:

```shell
/usr/local/lib/pkgconfig/mysqlclient.pc
```

Remove the occurrance of "-fno-omit-frame-pointer". This file is read-only by default so you will need to change that first.

### Linux 

Ensure that you have installed libmysqlclient-dev for MySQL version 5.6 or greater:

```shell
sudo apt-get install libmysqlclient-dev
```

Please note that Ubuntu 14 defaults to including a version of MySQL client which will not compile with this package. Install MySQL client version 5.6 or greater manually. 

## Setup

Add the "Perfect-MySQL" project as a dependency in your Package.swift file:

```swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-MySQL.git", versions: Version(0,0,0)..<Version(10,0,0))
```

### Import

Fist and foremost, in any of the source files you intend to use with MySQL, import the required modules with: 

```swift
import PerfectLib
import MySQL
import PerfectHTTP
```

## Quick Start

### Access the Database

In order to access the database, first setup your credentials:

```swift
let testHost = "127.0.0.1"
let testUser = "test"
let testPassword = "password"
let testDB = "schema"

//Obviously change these details to a database and user you already have setup
```

There are two common ways to connect to MySQL. First, you can omit the schema, so that you can use a separate selector. This is handy if you have multiple schemas that your program can choose from. 

```swift

    func fetchData() {
		   
			  let dataMysql = MySQL() // Create an instance of MySQL to work with
		   
		    let connected = mysql.connect(host: testHost, user: testUser, password: testPassword)
		    
		    guard connected else {
		        // verify we connected successfully
		        print(mysql.errorMessage())
		        return
		    }
		    
		    defer {
		        mysql.close() //This defer block makes sure we terminate the connection once finished, regardless of the result
		    }
		    
		    //Choose the database to work with
		    guard dataMysql.selectDatabase(named: testDB) else {
				    Log.info(message: "Failure: \(dataMysql.errorCode()) \(dataMysql.errorMessage())")
				    return
		    }
		}
```

Alternatively, you can pass the database you would like to access into the connection and skip selection:

```swift

    func fetchData() {
    
		    let dataMysql = MySQL() // Create an instance of MySQL to work with
		    
		    let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)
		    
		    guard connected else {
		        // verify we connected successfully
		        print(mysql.errorMessage())
		        return
		    }
		    
		    defer {
		        mysql.close() //This defer block makes sure we terminate the connection once finished, regardless of the result
		    }
		}
```

### Create Tables

Choosing the database is great, but it is much more helpful to run queries, such as adding tables programmatically. Expanding on our connection example above, it is relatively simple to add a query: 

```swift
		
    func setupMySQLDB() {
		    
		    let dataMysql = MySQL() // Create an instance of MySQL to work with
		    
		    let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)
		    
		    guard connected else {
		        // verify we connected successfully
		        print(mysql.errorMessage())
		        return
		    }
		    
		    defer {
		        mysql.close() //This defer block makes sure we terminate the connection once finished, regardless of the result
		    }
		    
		   //Run Query to Add Tables
		   
		   
		}
```

### Run Queries

Getting data from your schema is essential. It’s relatively easy to do. After running a query, we simply need to save our data and then act on it. In our example below, we’re assuming we have a table called options with a row id, an option name (text) and an option value (text). 

```swift

    func fetchData() {
    
		    let dataMysql = MySQL() // Create an instance of MySQL to work with
		    
		    let connected = mysql.connect(host: testHost, user: testUser, password: testPassword, db: testDB)
		    
		    guard connected else {
		        // verify we connected successfully
		        print(mysql.errorMessage())
		        return
		    }
		    
		    defer {
		        mysql.close() //This defer block makes sure we terminate the connection once finished, regardless of the result
		    }
		    
				 // Run the Query (for example all rows in an options table)
        let querySuccess = mysql.query(statement: "SELECT option_name, option_value FROM options")
            // make sure the query worked
            guard querySuccess else {
            return
            }
            
        // Save the results to use during this session
        let results = mysql.storeResults()! //We can implicitly unwrap because of the guard on the querySuccess. You’re welcome to use an if-let here if you like. 

        var ary = [[String:Any]]() //Create an array of dictionaries to store our results in for use

        results.forEachRow { row in
            let optionName = getRowString(forRow: row[0]) //Store our Option Name, which would be the first item in the row, and therefore row[0].
            let optionName = getRowString(forRow: row[1]) //Store our Option Value


            ary.append("\(optionName)":optionValue]) //store our options
        }
		}
```

## MySQL Server API

The MySQL server API provides you with a set of tools to connect to and work with MySQL server instances. This includes basic connections, disconnections, querying the instance for databases/tables, and running queries (which is actually a light wrapper for the full [Statements API](#mysql-statements-api). Results are returned, however, they handled and manipulated with the [Results API](#mysql-results-api). Statements also have their own [Satements API](#mysql-statements-api) that lets you work with statements in much more detail that simply running queries though the main MySQL class.

### init

```swift
public init()
```

Creates an instance of the MySQL class that allows you to interact with MySQL databases. 

### close

```swift
public func close()
```

Closes a connection to MySQL. Most commonly used as a defer after guarding a connection, making sure that your session will close no matter what the outcome.

### clientInfo

```swift
public static func clientInfo() -> String
```

This will give you a string of the MySQL client library version. i.e. "5.7.x" or similar depending on your MySQL installation. 

### errorCode & errorMessage

```swift
public func errorCode() -> UInt32
```

```swift
public func errorMessage() -> String
```

Error codes and messages are useful when debugging. These functions retrieve, display, and make use of those in Swift. You  can learn more about what those mean [here](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html). This is especially useful after connecting or running queries. Example:

```swift
let mysql = MySQL()
let connected = mysql.connect(host: dbHost, user: dbUser, password: dbPassword, db: dbName)
            guard connected else {
                // verify we connected successfully
                print(mysql.errorMessage())
                return
            }
```

In this case, the console output would print any error messages that came up during a connection failure. 

### serverVersion

```swift
public func serverVersion() -> Int
```

Returns an integer representation of the MySQL server’s version. 

### connect

```swift
public func connect(host hst: String? = nil, user: String? = nil, password: String? = nil, db: String? = nil, port: UInt32 = 0, socket: String? = nil, flag: UInt = 0) -> Bool
```

Opens a connection to the MySQL database when supplied with the bare minimum credentials for your server (Usually a host, user, &amp; password). Optionally, you can also specify the port, database, or socket. Specifying the schema is not required, as you can use the [selectDatabase()](#selectDatabase) method after the connection has been made. 

### selectDatabase

```swift
public func selectDatabase(named namd: String) -> Bool
```

Selects a database from the active MySQL connection. 

### listTables

```swift
public func listTables(wildcard wild: String? = nil) -> [String]
```

Returns an array of strings representing the different tables available on the selected database. 

### listDatabases

```swift
public func listDatabases(wildcard wild: String? = nil) -> [String]
```

Returns an array of strings representing the databases available on the MySQL server currently connected. 

### commit

```swift
public func commit() -> Bool
```

Commits the transaction. 

### rollback

```swift
public func rollback() -> Bool
```

Rolls back the transaction. 

### moreResults

### nextResult

### query

### storeResults

### setOption

### setOption with boolean

### setOption with interger

### setOption with string

## MySQL Results API

The results API set provides a set of tools for working with result sets that are obtained from running queries. 

### close

### dataSeek

### numRows

### numFields

### next

### forEachRow

## MySQL Statements API

### init

### close

### reset

### clearBinds

### freeResult

### errorCode & errorMessage

```swift
public func errorCode() -> UInt32
```

```swift
public func errorMessage() -> String
```

Error codes and messages are useful when debugging. These functions retrieve, display, and make use of those in Swift. You  can learn more about what those mean [here](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html). This is especially useful after connecting or running queries. Example:

```swift
let mysql = MySQL()
let connected = mysql.connect(host: dbHost, user: dbUser, password: dbPassword, db: dbName)
            guard connected else {
                // verify we connected successfully
                print(mysql.errorMessage())
                return
            }
```

In this case, the console output would print any error messages that came up during a connection failure. 

### prepare

### execute

### results

### fetch

### numRows

### affectedRows

### insertId

### fieldCount

### nextResult

### dataSeek

### paramCount

### bindParam