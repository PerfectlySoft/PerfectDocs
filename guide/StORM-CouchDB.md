# CouchDBStORM

## Relevant Examples

* [CouchDBStORM-Demo](https://github.com/PerfectExamples/CouchDBStORM-Demo)
* [Perfect-Session-CouchDB-Demo](https://github.com/PerfectExamples/Perfect-Session-CouchDB-Demo)
* [Perfect-Turnstile-CouchDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-CouchDB-Demo)


## Including in your project

When including the dependency in your project's Package.swift dependencies, you will have access to all nested dependencies including the database connector.

``` swift
.Package(url: "https://github.com/SwiftORM/CouchDB-Storm.git", majorVersion: 3)
```


## Creating a connection to your database

In order to connect to your database you will need to specify certain information.

``` swift
CouchDBConnection.host = "localhost"
CouchDBConnection.username = "username"
CouchDBConnection.password = "secret"
CouchDBConnection.port = 5984
CouchDBConnection.ssl = true
```

Once your connection information is specified it can be used in the object class to create the connection on demand.

``` swift
let obj = User()
```

## CouchDBStORM supported methods

### Connecting

`CouchDBConnection` - Sets the connection parameters for the CouchDB server access. Host, username and password information need to be specified. Port and SSL status need only be specified if different from the default (SSL false, and port 5984).

### Creating databases

The working database for the object is set by embedding this function in the object, and changing the name to the desired database name:

``` swift
	override open func database() -> String {
		return "mydbname"
	}
```

`setup()` - Creates the database.

> **NOTE:** The **primary key** is first property defined in the class.

### Saving objects

`save(rev: String = "")` - Saves object. If an ID has been defined, save() will perform an update, otherwise a new document is created. Takes an optional "rev" parameter which is the document revision to be used. If empty the object's stored _rev property is used.

`save(rev: String = "") {id in ... }` - Saves object. If an ID has been defined, save() will perform an update, otherwise a new document is created. Takes an optional "rev" parameter which is the document revision to be used. If empty the object's stored _rev property is used. The closure returns the new id if created.

`create()` - Forces the creation of a new database object. The revision property is also set once the save has been completed.

### Retrieving data

`get()` - Retrieves the document. Assumes id has been set in the object.

`get(String)` - Retrieves a document with a specified id.

`find([String: Any])` - Performs a find using the selector syntax. For example, `try find(["username":"joe"])` will find all documents that have a username equal to "joe". Full selector syntax can be found at [http://docs.couchdb.org/en/2.0.0/api/database/find.html#find-selectors](http://docs.couchdb.org/en/2.0.0/api/database/find.html#find-selectors)
		
Additionally the `find` can include:

*  `cursor: StORMCursor` - The optional `cursor` object is a [StORMCursor](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md)

### Deleting objects

`delete()` - Deletes the current object. Assumes the id and _rev properties are set.


