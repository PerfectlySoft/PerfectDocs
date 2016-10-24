# MySQLStORM

### Including in your project

When including the dependancy in your project's Package.swift dependancies, you will have access to all nested dependancies including the database connector.

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM", majorVersion: 0, minor: 0)
```


## Creating a connection to your database

In order to connect to your database you will need to specify certain information.

``` swift
connect = MySQLConnect(
	host: "localhost",
	username: "my_username",
	password: "my_password",
	database: "your_database",
	port: 3306
)
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