# MySQL

The MySQL connector provides a wrapper around MySQL, allowing interaction between your Perfect Applications and MySQL databases. 

### System Requirements

### macOS

Requires the use of Homebrew’s MySQL. 

```
brew install mysql@5.7
```

If you need Homebrew, you can install it with: 

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Unfortunately, at this point in time you will need to edit the mysqlclient.pc file located here:

```
/usr/local/lib/pkgconfig/mysqlclient.pc
```

Remove the occurrence of "-fno-omit-frame-pointer". This file is read-only by default so you will need to change its permissions first.

⚠️**NOTE**⚠️ The last mysql homebrew version that Perfect currently supports is 5.7, so please try this command when some missing type issues were found on your build:

```
$ brew install mysql@5.7 && brew link mysql@5.7 --force
```

### Linux 

Ensure that you have installed libmysqlclient-dev for MySQL version 5.6 or greater:

```
sudo apt-get install libmysqlclient-dev
```

Please note that Ubuntu 14 defaults to including a version of MySQL client which will not compile with this package. Install MySQL client version 5.6 or greater manually. 

### Setup

Add the "Perfect-MySQL" project as a dependency in your Package.swift file:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-MySQL.git", majorVersion: 3)
```

### Import

First and foremost, in any of the source files you intend to use with MySQL, import the required module with: 

``` swift
import PerfectMySQL
```

### Quick Start

### Access the Database

In order to access the database, set up your credentials:

``` swift
let testHost = "127.0.0.1"
let testUser = "test"
let testPassword = "password"
let testDB = "schema"

//Obviously change these details to a database and user you have already defined
```

There are two common ways to connect to MySQL. First, you can omit the schema, so that you can use a separate selector. This is handy if you have multiple schemas that your program can choose: 

``` swift

    func fetchData() {
		   
	            let mysql = MySQL() // Create an instance of MySQL to work with
		   
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
		    guard mysql.selectDatabase(named: testDB) else {
				    Log.info(message: "Failure: \(mysql.errorCode()) \(dataMysql.errorMessage())")
				    return
		    }
		}
```

Alternatively, you can pass the database you would like to access into the connection and skip selection:

``` swift

    func fetchData() {
    
		    let mysql = MySQL() // Create an instance of MySQL to work with
		    
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

``` swift
		
    func setupMySQLDB() {
		    
		    let mysql = MySQL() // Create an instance of MySQL to work with
		    
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
		  	let sql = """
		  	CREATE TABLE IF NOT EXISTS ticket (
    		id VARCHAR(64) PRIMARY KEY NOT NULL,
			expiration INTEGER)
    		"""
			guard mysql.query(statement: sql) else {
		        // verify the table was created successfully
		        print(mysql.errorMessage())
		        return
		   }
		   
		}
```

### Run Queries

Getting data from your schema is essential, and relatively easy to do. After running a query, save your data and then act on it. In the example below, we’re assuming we have a table called options with a row id, an option name (text) and an option value (text):

``` swift

    func fetchData() {
    
		    let mysql = MySQL() // Create an instance of MySQL to work with
		    
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
            let optionValue = getRowString(forRow: row[1]) //Store our Option Value


            ary.append("\(optionName)":optionValue]) //store our options
        }
	}
```

## MySQL Server API

The MySQL server API provides you with a set of tools to connect to and work with MySQL server instances. This includes basic connections, disconnections, querying the instance for databases/tables, and running queries (which is actually a light wrapper for the full [Statements API](#mysql-statements-api)). Results returned, however, are handled and manipulated with the [Results API](#mysql-results-api). Statements also have a [Statements API](#mysql-statements-api) that lets you work with statements in much more detail than simply running queries though the main MySQL class.

### init

``` swift
public init()
```

Creates an instance of the MySQL class that allows you to interact with MySQL databases. 

### NOTE for ⚠️Character Encoding⚠️

If your dataset contains non-ascii characters, please set this option for proper encoding:

``` swift
setOption(.MYSQL_SET_CHARSET_NAME, "utf8")
```

### close

``` swift
public func close()
```

Closes a connection to MySQL. Most commonly used as a defer after guarding a connection, making sure that your session will close no matter what the outcome.

### ping

MySQL connection will go away when idle timeout. The `ping()` function
can confirm the connectivity and also reconnect if need.

``` swift
guard mysql.ping() else {
	// connection lost
}
```

### clientInfo

``` swift
public static func clientInfo() -> String
```

This will give you a string of the MySQL client library version, e.g. "5.7.x" or similar depending on your MySQL installation.

### errorCode & errorMessage

``` swift
public func errorCode() -> UInt32
```

``` swift
public func errorMessage() -> String
```

Error codes and messages are useful when debugging. These functions retrieve, display, and make use of those in Swift. You  can learn more about what those mean [here](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html). This is especially useful after connecting or running queries. Example:

``` swift
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

``` swift
public func serverVersion() -> Int
```

Returns an integer representation of the MySQL server’s version. 

### connect

``` swift
public func connect(host hst: String? = nil, user: String? = nil, password: String? = nil, db: String? = nil, port: UInt32 = 0, socket: String? = nil, flag: UInt = 0) -> Bool
```

Opens a connection to the MySQL database when supplied with the bare minimum credentials for your server (usually a host, user, and password). As an option, you can specify the port, database, or socket. Specifying the schema is not required, as you can use the [selectDatabase()](#selectDatabase) method after the connection has been made. 

### selectDatabase

``` swift
public func selectDatabase(named namd: String) -> Bool
```

Selects a database from the active MySQL connection. 

### listTables

``` swift
public func listTables(wildcard wild: String? = nil) -> [String]
```

Returns an array of strings representing the different tables available on the selected database. 

### listDatabases

``` swift
public func listDatabases(wildcard wild: String? = nil) -> [String]
```

Returns an array of strings representing the databases available on the MySQL server currently connected. 

### commit

``` swift
public func commit() -> Bool
```

Commits the transaction. 

### rollback

``` swift
public func rollback() -> Bool
```

Rolls back the transaction. 

### moreResults

``` swift
public func moreResults() -> Bool
```

Checks `mysql_more_results` to see if any more results exist. 

### nextResult

``` swift
public func nextResult() -> Int
```

Returns the next result in a multi-result execution. Most commonly used in a while loop to produce an effect similar to running forEachRow(). For example:

``` swift
    var results = [[String?]]()

    while let row = results?.next() {
        results.append(row)

    }
```

### query

``` swift
public func query(statement stmt: String) -> Bool
```

Runs an SQL Query given as a string. 

### storeResults

``` swift
public func storeResults() -> MySQL.Results?
```

This retrieves a complete result set from the server and stores it on the client. This should be run after your query and before a function like forEachRow() or next() so that you can ensure that you iterate through all results.

### setOption

``` swift
public func setOption(_ option: MySQLOpt) -> Bool
public func setOption(_ option: MySQLOpt, _ b: Bool) -> Bool
public func setOption(_ option: MySQLOpt, _ i: Int) -> Bool
public func setOption(_ option: MySQLOpt, _ s: String) -> Bool
```

Sets the options for connecting and returns a Boolean for success or failure. Requires a MySQLOpt and has several versions to support setting options that require Booleans, integers, or strings as values. 

MySQLOpt values that are available to use are defined by the following enumeration:

``` swift
public enum MySQLOpt {
	case MYSQL_OPT_CONNECT_TIMEOUT, MYSQL_OPT_COMPRESS, MYSQL_OPT_NAMED_PIPE,
		MYSQL_INIT_COMMAND, MYSQL_READ_DEFAULT_FILE, MYSQL_READ_DEFAULT_GROUP,
		MYSQL_SET_CHARSET_DIR, MYSQL_SET_CHARSET_NAME, MYSQL_OPT_LOCAL_INFILE,
		MYSQL_OPT_PROTOCOL, MYSQL_SHARED_MEMORY_BASE_NAME, MYSQL_OPT_READ_TIMEOUT,
		MYSQL_OPT_WRITE_TIMEOUT, MYSQL_OPT_USE_RESULT,
		MYSQL_OPT_USE_REMOTE_CONNECTION, MYSQL_OPT_USE_EMBEDDED_CONNECTION,
		MYSQL_OPT_GUESS_CONNECTION, MYSQL_SET_CLIENT_IP, MYSQL_SECURE_AUTH,
		MYSQL_REPORT_DATA_TRUNCATION, MYSQL_OPT_RECONNECT,
		MYSQL_OPT_SSL_VERIFY_SERVER_CERT, MYSQL_PLUGIN_DIR, MYSQL_DEFAULT_AUTH,
		MYSQL_OPT_BIND,
		MYSQL_OPT_SSL_KEY, MYSQL_OPT_SSL_CERT,
		MYSQL_OPT_SSL_CA, MYSQL_OPT_SSL_CAPATH, MYSQL_OPT_SSL_CIPHER,
		MYSQL_OPT_SSL_CRL, MYSQL_OPT_SSL_CRLPATH,
		MYSQL_OPT_CONNECT_ATTR_RESET, MYSQL_OPT_CONNECT_ATTR_ADD,
		MYSQL_OPT_CONNECT_ATTR_DELETE,
		MYSQL_SERVER_PUBLIC_KEY,
		MYSQL_ENABLE_CLEARTEXT_PLUGIN,
		MYSQL_OPT_CAN_HANDLE_EXPIRED_PASSWORDS
}
```

## MySQL Results API

The results API set provides a set of tools for working with result sets that are obtained from running queries. 

### close

``` swift
public func close()
```

Closes the result set by releasing the results. Make sure you have a close function that is always executed after each session with a connection, otherwise you are going to have some memory problems. This is most commonly found in the defer block, just like in the examples at the top of this page.

### dataSeek

``` swift
public func dataSeek(_ offset: UInt)
```

Moves to an arbitrary row number in the results given an unsigned integer as an offset. 

### numRows

``` swift
public func numRows() -> Int
```

Lets you know how many rows are in a result set.

### numFields

``` swift
public func numFields() -> Int
```

Similar to numRows, but returns the number of columns in a result set instead. Very useful for `Select *` type queries where you may need to know how many columns are in the results.

### next

``` swift
public func next() -> Element?
```

Returns the next row in the result set as long as one exists.

### forEachRow

``` swift
public func forEachRow(callback: (Element) -> ())
```

Iterates through all rows in query results. Most useful for appending elements to an array or dictionary, just like we did in the [quick start guide](#Quick-Start).

### MySQL Statements API

### init

``` swift
public init(_ mysql: MySQL)
```

Initializes the MySQL statement structure. This is very commonly used by other API functions to create a statement structure after you’ve passed in a string.

### close

``` swift
public func close()
```

This frees the MySQL statement structure pointer. Use it or lose valuable memory to the underlying MySQL C API. Most commonly in a defer block, just like we used in [quick start](#Quick-Start).  

### reset

``` swift
public func reset()
```

Resets the statement buffers that are in the server. This doesn’t affect bindings or stored result sets. Learn more about this feature [here](https://dev.mysql.com/doc/refman/5.7/en/mysql-stmt-reset.html).

### clearBinds

``` swift
func clearBinds()
```

Clears the current bindings. 

### freeResult

``` swift
public func freeResult(
```

Releases memory tied up in with the result set produced by execution of a prepared statement. Also closes a cursor if one is open for the statement. 

### errorCode & errorMessage

``` swift
public func errorCode() -> UInt32
```

``` swift
public func errorMessage() -> String
```

Error codes and messages are useful when debugging. These functions retrieve, display, and make use of both in Swift. You can learn more about what those mean [here](https://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html). This is especially useful after connecting or running queries. Example:

``` swift
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

``` swift
public func prepare(statement query: String) -> Bool
```

Prepares a SQL statement for execution. More commonly called by other functions in the API, but public if you need it. 

### execute

``` swift
public func execute() -> Bool
```

Executes a prepared statement. 

### results

``` swift
public func results() -> MySQLStmt.Results
```

Returns current results from the server. 

### fetch

``` swift
public func fetch() -> FetchResult
```

Fetches the next row of data from the result set.

### numRows

``` swift
public func numRows() -> UInt
```

Returns the row count in a buffered statement result set.

### affectedRows

``` swift
public func affectedRows() -> UInt
```

Returns the number of rows that were changed, deleted, or inserted by a prepared statement or an UPDATE, DELETE, or INSERT statement that was executed.

### insertId

``` swift
public func insertId() -> UInt
```

Returns the row id number for the last row inserted by a prepared statement, as long as id was an auto-increment enabled column. 

### fieldCount

``` swift
public func fieldCount() -> UInt
```

Returns the number of columns in the results for the most recently executed statement. 

### nextResult

``` swift
public func nextResult() -> Int
```

Returns the next result in a multi-result execution. 

### dataSeek

``` swift
public func dataSeek(offset: Int)
```

Given an offset, it will seek to an arbitrary row in a statement result set.

### paramCount

``` swift
public func paramCount() -> Int
```

Returns the number of parameters in a prepared statement. 

### bindParam

``` swift
public func bindParam()
func bindParam(_ s: String, type: enum_field_types)
public func bindParam(_ d: Double)
public func bindParam(_ i: Int)
public func bindParam(_ i: UInt64)
public func bindParam(_ s: String)
public func bindParam(_ b: UnsafePointer<Int8>, length: Int)
public func bindParam(_ b: [UInt8])
```

Variations above on the bindParam() allow binding to statement parameters with different types. If no arguments are passed, it creates a null binding. 
