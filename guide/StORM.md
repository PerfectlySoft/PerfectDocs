# Swift ORM: "StORM"

StORM is a modular ORM for Swift, layered on top of [Perfect](https://github.com/PerfectlySoft/Perfect).

It aims to be easy to use, but flexible, and maintain consistency between datasource implementations for the user: you, the developer. It tries to allow you to write great code without worrying about the details of how to interact with the database.

## Relevant Examples

PostgresSQL

* [PostgreStORM-Demo](https://github.com/PerfectExamples/PostgreStORM-Demo)
* [Perfect-Session-PostgreSQL-Demo](https://github.com/PerfectExamples/Perfect-Session-PostgreSQL-Demo)
* [Perfect-Turnstile-PostgreSQL-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo)

MySQL:

* [MySQLStorm-Demo](https://github.com/PerfectExamples/MySQLStorm-Demo)
* [Perfect-Turnstile-MySQL-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MySQL-Demo)
* [Perfect-Session-MySQL-Demo](https://github.com/PerfectExamples/Perfect-Session-MySQL-Demo)

CouchDB: 

* [CouchDBStORM-Demo](https://github.com/PerfectExamples/CouchDBStORM-Demo)
* [Perfect-Session-CouchDB-Demo](https://github.com/PerfectExamples/Perfect-Session-CouchDB-Demo)
* [Perfect-Turnstile-CouchDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-CouchDB-Demo)

MongoDB:

* [MongoDBStORM-Demo](https://github.com/PerfectExamples/MongoDBStORM-Demo)
* [Perfect-Session-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Session-MongoDB-Demo)
* [Perfect-Turnstile-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MongoDB-Demo)

SQLite

* [Perfect-Turnstile-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo)
* [Perfect-Session-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Session-SQLite-Demo)

## StORM Documentation topics

[Setting up a class:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Setting-up-a-class.md) How to create a class that inherits all the StORM functionality.

[Saving, Retrieving and Deleting Rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Saving-Retrieving-and-Deleting-Rows.md) Basic database operations

[StORMCursor:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md) Managing found sets for result set pagination.

[Inserting rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Insert.md) More detailed access to inserting rows.

[Updating rows:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Update.md) More detailed access to the update process.

[Lifecycle events:](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORMLifecycleEvents.md) Details and examples of the global StORM lifecycle events.


## Datasource specific documentation for StORM

* [SQLite3](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-SQLite.md)
* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-PostgreSQL.md)
* [MySQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-MySQL.md)
* [Apache CouchDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-CouchDB.md)
* [MongoDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-MongoDB.md)


### Including in your project

When including the dependency in your project's Package.swift dependencies, you will have access to all nested dependencies including the database connector.

To include PostgresStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/Postgres-StORM.git", majorVersion: 3)
```

To include MySQLStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM.git", majorVersion: 3)
```

To include SQLiteStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 3)
```

To include CouchDBStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/CouchDB-StORM.git", majorVersion: 3)
```

To include MongoDBStORM:

``` swift
.Package(url: "https://github.com/SwiftORM/MongoDB-StORM.git", majorVersion: 3)
```

Remember: after you change your `Package.swift` file, you need to regenerate your Xcode Project file using:

```
swift package generate-xcodeproj 
```

