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

Getting data from your schema is essential. It’s relatively easy to do. After running a query, we simply need to save our data and then act on it:

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

## Full API Reference

