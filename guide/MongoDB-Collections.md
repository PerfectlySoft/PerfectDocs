# MongoDB Collections
Once your connection and database are defined, you will be working with a Collection to create, update, delete and query documents within that collection.

A collection can be defined either by "cascading" through defining the connection client, the database then the collection:

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
let db = client.getDatabase(name: "test")
let collection = db.getCollection(name: "testcollection")
```
or by assigning directly after opening the connection:

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
let collection = MongoCollection(
	client: client, 
	databaseName: "test", 
	collectionName: "testcollection"
	)
```
### Closing connections
Remember to set up your connections to close:

``` swift
defer {
    collection.close()
    db.close() 			// if created by "cascade"
    client.close()
}
```

### Collection Name

To return the collection name as a string:

``` swift
collection.name()
```

### Rename collection

To rename the collection using newDbName and newCollectionName, with an option to drop existing collection immediately instead of after the move:

``` swift
collection.rename(
	newDbName: <String>, 
	newCollectionName: <String>,
	dropExisting: <Bool>
	)
```

#### Parameters
* **newDbName:** String name for db after move
* **newCollectionName:** String name for collection after move
* **dropExisting:** Bool option to drop existing collection immediately instead of after move

The returned value is the status of the renaming action.

### Drop collection

```.drop``` removes a collection from the database. The method also removes any indexes associated with the dropped collection.

``` swift
collection.drop()
```

The returned value is the status of the drop action.


### Inserting a document into a collection

Insert document into the current collection returning a result status:

``` swift
collection.insert(_ 
	document: <BSON>,
	flag: <MongoInsertFlag>
	)
```
#### Parameters
* **document:** BSON document to be inserted
* **flag:** Optional MongoInsertFlag defaults to .None

The returned value is the status of the insert action.

#### MongoInsertFlag options

The MongoInsertFlag enum has the following options:

* **None:** No additional action is to be taken. This is the default option.
* **ContinueOnError:** Instruct MongoDB to ignore errors.
* **NoValidate:** Ignore validation process.

### Updating a document
To update a document in the collection, assemble the BSON object to replace the existing document, and the selector. 

``` swift
collection.update( 
	update: <BSON>,
	selector: <BSON>,
	flag: <MongoUpdateFlag>
	)
```
#### Parameters
* **update:** BSON document to replace existing document with
* **selector:** BSON document with selection criteria
* **flag:** Optional MongoUpdateFlag defaults to .None

The returned value is the status of the update action.

#### MongoUpdateFlag options

The MongoUpdateFlag enum has the following options:

* **None:** No additional action is to be taken. This is the default option.
* **Upsert:** Insert, or if selector matches a record, update.
* **MultiUpdate:** Update more than one document id matched by the selector.
* **NoValidate:** Ignore validation process.

### Save

Updates an existing document or inserts a new document, depending on its document parameter. 

``` swift
collection.save(document: <BSON>)
```
#### Parameters
* **document:** BSON document to save

The returned value is the status of the save action.

* If the document does not contain an ```_id``` field, a new document will be created.
* If an ```_id``` is specified, ```save``` will perform an "upsert": If a matching ```_id``` is found in the collection an update will occur, otherwise an insert will be performed

``` swift
let bson = BSON()
defer {
	bson.close()
}
	
bson.append(key: "stringKey", string: "String Value")
bson.append(key: "intKey", int: 42)
	
let result2 = collection.save(document: bson)

```

### Find

Selects documents in a collection and returns a cursor to the selected documents.

``` swift
collection.find( 
	query: <BSON>,
	fields: <BSON>, 
	flags: <MongoQueryFlag>, 
	skip: <Int>, 
	limit: <Int>, 
	batchSize: <Int>
	)
```
#### Parameters
* **query:** *(Optional)* Specifies selection filter using query operators. To return all documents in a collection, omit this- Parameter or pass an empty document ({}).
* **fields:** *(Optional)* Specifies the fields to return in the documents that match the query filter. To return all fields in the matching documents, omit this parameter.
* **flags:** *(Optional)*  Set queryFlags for the current search
* **skip:** *(Optional)*  Skip the supplied number of records
* **limit:** *(Optional)*  Return no more than the supplied number of records
* **batchSize:** *(Optional)*  Change number of automatically iterated documents

#### Return Value
A cursor to the documents that match the query criteria. When the find() method "returns documents" the method is actually returning a cursor to the documents.

### Find and Modify

...

``` swift
collection.findAndModify(
	query: <BSON>, 
	sort: <BSON>, 
	update: <BSON>, 
	fields: <BSON>, 
	remove: <Bool>, 
	upsert: <Bool>, 
	new: <Bool>
	)
```
#### Parameters
* **query:** *Optional*. The selection criteria for the modification. The query field employs the same query selectors as used in the `db.collection.find()` method. Although the query may match multiple documents, `findAndModify()` will only select one document to modify.
* **sort:** *Optional*. Determines which document the operation modifies if the query selects multiple documents. `findAndModify()` modifies the first document in the sort order specified by this argument.
* **update:** Must specify either the remove or the update field. Performs an update of the selected document. The update field employs the same update operators or field: value specifications to modify the selected document.
* **fields:** *Optional*. A subset of fields to return. The fields document specifies an inclusion of a field with 1, as in: `fields: { : 1, : 1, … }`.
* **remove:** Must specify either the remove or the update field. Removes the document specified in the query field. Set this to true to remove the selected document. The default is false.
* **upsert:** *Optional*. Used in conjunction with the update field. When true, `findAndModify()` creates a new document if no document matches the query, or if documents match the query, `findAndModify()` performs an update. To avoid multiple upserts, ensure that the query fields are uniquely indexed. The default is false.
* **new:** *Optional*. When true, returns the modified document rather than the original. The `findAndModify()` method ignores the new option for remove operations. The default is false.

### Count

### Deleting

### Creating an Index

### Dropping an Index

### Stats

### GetLastError