# Swift ORM: "StORM"

StORM is a modular ORM for Swift, layered on top of [Perfect](https://github.com/PerfectlySoft/Perfect)

It aims to be easy to use, but flexible, and maintain consistency between datasource implementations for the user: you, the developer. It tries to allow you write great code without worrying about the details of how to interact with the database.

## StORM Documentation topics:

[Setting up a class:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Setting-up-a-class.md) How to create a class that inherits all the StORM functionality.

[Saving, Retrieving and Deleting Rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Saving-Retrieving-and-Deleting-Rows.md) Basic database operations

[StORMCursor:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md) Managing found sets for result set pagination.

[Inserting rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Insert.md) More detailed access to inserting rows.

[Updating rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Update.md) More detailed access to the update process.


## Datasource specific documentation for StORM:

* [SQLite3](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-SQLite.md)
* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-PostgreSQL.md)
* [MySQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-MySQL.md)
* [Apache CouchDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-CouchDB.md)


### Including in your project

When including the dependancy in your project's Package.swift dependancies, you will have access to all nested dependancies including the database connector.

For example to include PostgresStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/Postgres-StORM.git", majorVersion: 1, minor: 0)
```

For example to include MySQLStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM.git", majorVersion: 1, minor: 0)
```

To include SQLiteStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 1, minor: 0)
```

To include CouchDBStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/CouchDB-StORM.git", majorVersion: 1, minor: 0)
```

Remember after you change your `Package.swift` file, you need to regenerate your Xcode Project file using:

```
swift package generate-xcodeproj 
```

