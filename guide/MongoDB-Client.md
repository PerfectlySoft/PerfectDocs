# MongoDB Client

The MongoClient class is where the initial connection to the MongoDB server is defined. 

Create new Mongo client connection:

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
```

### Closing the Connection

Once the connection is established and the database and collections have been defined, set the connection to close once completed using `defer`. This is done in reverse order - close collections, then databases, and then finally the client connection.

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```

### Create Database Reference

`getDatabase` returns the specified MongoDatabase using the current connection.

``` swift
let db = client.getDatabase(
	databaseName: <String>
	)
```

#### Parameters

* **databaseName:** String name of database to be used

### Create Collection Reference

`getCollection` returns the specified MongoCollection from the specified database using the current connection.

``` swift
let collection = client.getCollection(
	databaseName: <String>, 
	collectionName: <String>
	)
```

#### Parameters

* **databaseName:** String name of database to be used
* **collectionName:** String name of collection to be retrieved

### Get Current Mongo Server Status

`serverStatus` returns: a Result object representing the server status.

``` swift
let status = client.serverStatus()
```

### Return String Array of Current Database Names

Use `databaseNames` to build a string array of current database names:


``` swift
let dbnames = client.databaseNames()
```

