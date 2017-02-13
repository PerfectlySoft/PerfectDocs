# MongoDB Database

Use the MongoDB Database class to create a reference to a named database using a provided MongoClient instance.

Create a new Mongo Database connection:

``` swift
let database = try! MongoDatabase(
	client: <MongoClient>, 
	databaseName: <String>
	)
```

### Closing the Connection

Once the connection is established and the database and collections have been defined, set the connection to close once completed using `defer`. This is done in reverse order: close collections, then databases, then finally the client connection.

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```

### Drop the Current Database

Drops the current database, deleting the associated data files.

``` swift
database.drop()
```

### Current Database Name

`name()` returns the name of the current database.

``` swift
let name = database.name()
```

### Create a New Collection

``` swift
database.createCollection(name: <String>, options: <BSON>)
```

#### Parameters

* **name:** String, name of collection to be created
* **options:**  BSON document listing options for new collection

### Create Reference to MongoDB Collection Referenced by Name

Use `getCollection` to create a reference to a MongoCollection:


``` swift
let collection = database.getCollection(name: <String>)
```

### Create String Array of Current Database Collections' Names

Use `collectionNames` to create an array of the databases' collection names:


``` swift
let collection = database.collectionNames()
```

