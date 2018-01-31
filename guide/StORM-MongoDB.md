# MongoDBStORM

## Relevant Examples

* [MongoDBStORM-Demo](https://github.com/PerfectExamples/MongoDBStORM-Demo)
* [Perfect-Session-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Session-MongoDB-Demo)
* [Perfect-Turnstile-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MongoDB-Demo)


## Including in your project

When including the dependency in your project's Package.swift dependencies, you will have access to all nested dependencies including the database connector.

``` swift
.Package(url: "https://github.com/SwiftORM/MongoDB-Storm.git", majorVersion: 3)
```

## Creating a connection to your database

In order to connect to your database you will need to specify certain information.

``` swift
MongoDBConnection.host = "localhost"
MongoDBConnection.port = 27017
MongoDBConnection.ssl = true
MongoDBConnection.database = "mydb"

// Authentication credentials.
// Only required if authModeType = .standard
MongoDBConnection.username = "username"
MongoDBConnection.password = "secret"
MongoDBConnection.authdb = "authenticationSource"

// Authentication mode: 
// .none (default), or .standard
MongoDBConnection.authModeType = .none
```

Once your connection information is specified it can be used in the object class to create the connection on demand.

``` swift
let obj = User()
```

## MongoDBStORM supported methods

### Connecting

`MongoDBConnection` - Sets the connection parameters for the MongoDB server access. Host, and database information need to be specified. Port and SSL status need only be specified if different from the default (SSL false, and port 27017).

### Creating collections

The working collection for the object is set by embedding this function in the object, and changing the name to the desired collection name:

``` swift
	override init() {
		super.init()
		_collection = "users_demo"
	}
```

Note that unlike other StORM implementations, it is not required to invoke a `setup()` function call to ensure the collection is created. MongoDB will create the collection by default if it does not exist at the first data insertion attempt.

> **NOTE:** The **primary key** is first property defined in the class.

### Saving objects

`save()` - Saves object. If an ID has been defined, save() will perform an update, otherwise a new document is created.

### Retrieving data

`get()` - Retrieves the document. Assumes id has been set in the object.

`get(String)` - Retrieves a document with a specified id.

`find([String: Any])` - Performs a find. For example, `try find(["username":"joe"])` will find all documents that have a username equal to "joe".
		
Additionally the `find` can include:

*  `cursor: StORMCursor` - The optional `cursor` object is a [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md)

### Deleting objects

`delete()` - Deletes the current object. Assumes the id property is set.


