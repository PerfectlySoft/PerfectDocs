# SQLiteStORM

### Including in your project

When including the dependancy in your project's Package.swift dependancies, you will have access to all nested dependancies including the database connector.

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 0, minor: 0)
```


## Creating a connection to your database

In order to connect to your database you will need to specify the path to the database file.

``` swift
connect = SQLiteConnect("./mydb")
```
Once your connection object is created it can be used in the object class to create the connection on demand.

``` swift
let obj = User(connect)
```
In the above code the user object is created and the connection object embedded via the init function.

However there are many cases where the user object will be created without the embedded connection. To add after it has been instantiated, simply assign the connection object to the `.connection` property:

``` swift
obj.connection = connect
```