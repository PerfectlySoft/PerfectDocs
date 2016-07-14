# MongoDB Database

Use the MongoDatabase class to create a reference to a named database using provided MongoClient instance. 

Create new Mongo Database connection:

``` swift
let database = try! MongoDatabase(
	client: <MongoClient>, 
	databaseName: <String>
	)
```

### Closing the connection

Once the connection is established and the database and collections have been defined, set the connection to close once completed using `defer`. This is done in reverse order - close collections, then databases, then finally the client connection.

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```

### Drop the current database

Drops the current database, deleting the associated data files.

``` swift
database.drop()
```


### Current database name

`name()` returns the name of the current database.

``` swift
let name = database.name()
```

### Create new Collection

`serverStatus` returns: a Result object representing the server status.

``` swift
databse.createCollection(name: <String>, options: <BSON>)
```

####Parameters

* **name:** String, name of collection to be created
* **options:**  BSON document listing options for new collection


### Create reference to MongoCollection referenced by name

Use `getCollection` to create a reference to a MongoCollection:


``` swift
let collection = database.getCollection(name: <String>)
```


### Create String Array of current database collections' names

Use `collectionNames` to create an array of the databases' collection names:


``` swift
let collection = database.collectionNames()
```

