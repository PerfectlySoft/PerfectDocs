# Apache CouchDB

The CouchDB connector provides access to Apache CouchDB servers. It allows you to search for, add, update and delete documents in a CouchDB server.


## System Requirements

This module uses libcurl along with the Perfect-CURL package.

### macOS

macOS currently includes suitable versions of the dependent libraries and no manual installation is necessary.

### Linux
Install the libcurl4-openssl-dev package through apt.

```
sudo apt-get install libcurl4-openssl-dev
```

## Setup
Add the "Perfect-CouchDB" project as a dependency in your Package.swift file:

``` swift
.Package(
    url:"https://github.com/PerfectlySoft/Perfect-CouchDB.git",majorVersion: 3)
```

### Import

In any of the source files where you intend to use with this module, add the following import:

``` swift
import PerfectCouchDB
```

## Overview

The main objects that you will be working with when accessing CouchDB databases are described here.

### CouchDB()

This is the main interface to a CouchDB server. The class has 4 publicly accessible properties:

* debug: Defines the level of console & log output. Default: false;
* database: The instance's current database;
* connector: The instance's connector information;
* authentication: A container for the authentication settings.

#### Server API Functions

* serverInfo() - returns the current server information
* serverActiveTasks() - returns the active tasks on the server
* listDatabases() - lists all accessible databases on the server

#### Database operations

To check if a database exists, use the`databaseExists(String)` method, which returns a true or false response.

``` swift
db.databaseExists("mydb")
```

Deleting a database is done via the `databaseDelete(String)` method. A `CouchDBResponse` object is returned containing the result of the operation.

``` swift
db.databaseDelete("mydb")
```

Geting database information is accomplished using `databaseInfo(String)`. The response is a tuple of `CouchDBResponse, [String:Any]` where CouchDBResponse is the status of the request (200 OK; 404 Not Found), and the [String:Any] are the name/value pairs of the response.

``` swift
db.databaseInfo("mydb")
```

Possible information responses include:

* committed\_update_seq (Int) – The number of committed update.
* compact_running (Bool) – Set to true if the database compaction routine is operating on this database.
* db_name (String) – The name of the database.
* disk\_format_version (Int) – The version of the physical format used for the data when it is stored on disk.
* data_size (Int) – The number of bytes of live data inside the database file.
* disk_size (Int) – The length of the database file on disk. Views indexes are not included in the calculation.
* doc_count (Int) – A count of the documents in the specified database.
* doc\_del_count (Int) – Number of deleted documents
* instance\_start_time (String) – Timestamp of when the database was opened, expressed in microseconds since the epoch.
* purge_seq (Int) – The number of purge operations on the database.
* update_seq (Int) – The current number of updates to the database.

#### Adding a document

To add a new document to the database, using the supplied JSON document structure, use the `addDoc` method.

If the JSON structure includes the \_id field, then the document will be created with the specified document ID. If the \_id field is not specified, a new unique ID will be generated, following whatever UUID algorithm is configured for that server.

Parameters:

* db: The database name
* doc: The document to be stored.

The stored document can be supplied as a JSON encoded string, or as a [String: Any] type.

This returns a tuple of CouchDBResponse and the returned JSON as [String:Any].

``` swift
let data = ["one":"ONE","two":"TWO"]
db.addDoc("mydb", doc: data)
```

Alternatively, you can use the `create()` method. This creates a new named document, or creates a new revision of the existing document. Unlike the `addDoc` method, you must specify the document ID in the request URL.

Parameters:

* db – Database name
* docid – Document ID
* doc - [String: Any] object to be stored as JSON

Response JSON Object:

* id (string) – Document ID
* ok (boolean) – Operation status
* rev (string) – Revision MVCC token

Status Codes:

* 201 Created – Document created and stored on disk
* 202 Accepted – Document data accepted, but not yet stored on disk
* 400 Bad Request – Invalid request body or parameters
* 401 Unauthorized – Write privileges required
* 404 Not Found – Specified database or document ID doesn’t exists
* 409 Conflict – Document with the specified ID already exists or specified revision is not latest for target document

``` swift
db.create("mydb", docid: String, doc: [String: Any])
```

#### Duplicating a specific document

The `copy` method copies an existing document to a new or existing document. The source document docid is specified, with the destination parameter specifying the target document.

Parameters:

* db – Database name
* docid – Document ID
* rev (string) – Revision to copy from. Optional
* destination – Destination document

Response JSON Object:

* id (string) – Document document ID
* ok (boolean) – Operation status
* rev (string) – Revision MVCC token

Status Codes:

* 201 Created – Document successfully created
* 202 Accepted – Request was accepted, but changes are not yet stored on disk
* 400 Bad Request – Invalid request body or parameters
* 401 Unauthorized – Read or write privileges required
* 404 Not Found – Specified database, document ID or revision doesn’t exist
* 409 Conflict – Document with the specified ID already exists or specified revision is not latest for target document

``` swift
db.copy("mydb", docid: String, doc: [String:Any], rev: String, destination: String)
```

#### Searching for documents

The `filter` method returns a JSON structure of all of the documents in a given database. The information is returned as a JSON structure containing meta information about the return structure, including a list of all documents and basic contents, consisting the ID, revision and key. The key is the from the document’s \_id.

Parameters:

* db – Database name
* queryParams - CouchDBQuery object.

Response JSON Object:

* offset (Int) – Offset where the document list started
* rows (Array) – Array of view row objects. By default the information returned contains only the document ID and revision.
* total_rows (Int) – Number of documents in the database/view. Note that this is not the number of rows returned in the actual query.
* update_seq (Int) – Current update sequence for the database

``` swift
let query = CouchDBQuery()
// See the CouchDBQuery section for configuration options
db.filter("mydb", query)
```

Alternatively, you can use the `find` method. This differs from `filter` in the options supplied to the API. `find` uses a declarative JSON querying syntax.

Parameters supported by the CouchDBQuery in this context:

* selector ([String:Any]) – JSON object describing criteria used to select documents. More information provided in the section on selector syntax.
* limit (Int) – Maximum number of results returned. Default is 25. Optional
* skip (Int) – Skip the first ‘n’ results, where ‘n’ is the value specified. Optional
* sort ([String:Any]) – JSON array following sort syntax. Optional
* fields ([String:Any]) – JSON array specifying which fields of each object should be returned. If it is omitted, the entire object is returned. More information provided in the section on filtering fields. Optional
* use\_index ([String:Any]) – Instruct a query to use a specific index.

Response JSON Object:

* docs (object) – Documents matching the selector

Status Codes:

* 200 OK – Request completed successfully
* 400 Bad Request – Invalid request
* 401 Unauthorized – Read permission required
* 500 Internal Server Error – Query execution error

``` swift
let findObject = CouchDBQuery()
findObject.selector = data
findObject.limit = 10
findObject.skip = 0

do {
	let (code, response) = try db.find("mydb",queryParams: findObject)
} catch {
	print("\(error)")
}
```

#### Retrieving a specific document

To retrieve a specific document from a database, use the get() method.

This method returns document by the specified docid from the specified db. Unless you request a specific revision, the latest revision of the document will always be returned.

Parameters:

* db – Database name
* docid – Document ID

Query Parameters:

* attachments (Bool) – Includes attachments bodies in response. Default is false
*  att\_encoding_info (Bool) – Includes encoding information in attachment stubs if the particular attachment is compressed. Default is false.
* atts_since (array) – Includes attachments only since specified revisions. Doesn’t include attachments for specified revisions. Optional
* conflicts (Bool) – Includes information about conflicts in document. Default is false
* deleted_conflicts (Bool) – Includes information about deleted conflicted revisions. Default is false
* latest (Bool) – Forces retrieving latest “leaf” revision, no matter what rev was requested. Default is false
* local_seq (Bool) – Includes last update sequence for the document. Default is false
* meta (Bool) – Acts same as specifying all conflicts, deleted\_conflicts and open_revs query parameters. Default is false
* open_revs (array) – Retrieves documents of specified leaf revisions. Additionally, it accepts value as all to return all leaf revisions. Optional
* rev (String) – Retrieves document of specified revision. Optional
* revs (Bool) – Includes list of all known document revisions. Default is false
* revs_info (Bool) – Includes detailed information for all known document revisions. Default is false

Note that all these query params are optional apart from the document id (docid).
	
Response JSON Object:
	
* _id (string) – Document ID
* _rev (string) – Revision MVCC token
* _deleted (boolean) – Deletion flag. Available if document was removed
* _attachments (object) – Attachment’s stubs. Available if document has any attachments
* _conflicts (array) – List of conflicted revisions. Available if requested with conflicts=true query parameter
* \_deleted\_conflicts (array) – List of deleted conflicted revisions. Available if requested with deleted_conflicts=true query parameter
* \_local\_seq (string) – Document’s update sequence in current database. Available if requested with local_seq=true query parameter
* \_revs\_info (array) – List of objects with information about local revisions and their status. Available if requested with open_revs query parameter
* _revisions (object) – List of local revision tokens without. Available if requested with revs=true query parameter

Status Codes:

* 200 OK – Request completed successfully
* 304 Not Modified – Document wasn’t modified since specified revision
* 400 Bad Request – The format of the request or revision was invalid
* 401 Unauthorized – Read privilege required
* 404 Not Found – Document not found

``` swift
db.get(
		"mydb",
		docid: "docidString",
		attachments: false,
		att_encoding_info: false,
		atts_since: [String](),
		conflicts: false,
		deleted_conflicts: false,
		latest: false,
		local_seq: false,
		meta: false,
		open_revs: [String](),
		rev: "specificRevision",
		revs: false,
		revs_info: false
)
```

#### Update a document

To update an existing document you must specify the current revision number within the rev parameter.

Parameters:

* db – Database name
* docid – Document ID
* doc – Document body as [String: Any]
* rev – Document Revision

Response JSON Object:

* id (string) – Document document ID
* ok (boolean) – Operation status
* rev (string) – Revision MVCC token

``` swift 
db.update("mydb", docid: String, doc: [String:Any], rev: String)
```

#### Delete a document

The `delete` method marks the specified document as deleted by adding a field _deleted with the value true. Documents with this field will not be returned within requests anymore, but stay in the database. You must supply the current (latest) revision by using the rev parameter.

> Note: CouchDB doesn’t completely delete the specified document. Instead, it leaves a tombstone with very basic information about the document. The tombstone is required so that the delete action can be replicated across databases.

Parameters:

* db – Database name
* docid – Document ID
* rev - Actual document’s revision

Response JSON Object:

* id (string) – Document ID
* ok (boolean) – Operation status
* rev (string) – Revision MVCC token

Status Codes:

* 200 OK – Document successfully removed
* 202 Accepted – Request was accepted, but changes are not yet stored on disk
* 400 Bad Request – Invalid request body or parameters
* 401 Unauthorized – Write privileges required
* 404 Not Found – Specified database or document ID doesn’t exists
* 409 Conflict – Specified revision is not the latest for target document

``` swift
db.delete("mydb", docid: String, rev: String)
```

### CouchDBAuthentication()

This is the container struct for holding the CouchDB Authentication information.

* **authType**: The authentiction mode. .none, .basic, or .session
* **username**: Username for the authentication
* **password**: Password for the authentication
* **token**: The token obtained if authtype is .session. Irrelevant if auth type is not .session

An instance of the struct can be set up in the following ways:

``` swift
let auth = CouchDBAuthentication("username","pwd")
let auth = CouchDBAuthentication("username","pwd", auth: .basic)
```

Calculating the base64 token:

``` swift
let auth = CouchDBAuthentication("username","pwd")
let token = auth.basic()
```

### CouchDBConnector()

This struct holds the basic CouchDB connector information.

* **ssl**: Bool - SSL mode enabled/disabled. Default value: disabled (false)
* **host**: String - CouchDB Server hostname or IP address. Default: localhost
* **port**: Int - CouchDB Server port. Default: 5984

``` swift
var thisConnector = CouchDBConnector()
thisConnector.host = "192.168.0.12"
thisConnector.ssl = true
thisConnector.port = 5984
```

### CouchDBQuery()

CouchDBQuery is container class for query params used in the filter and find functions.

It has the following properties:

* **conflicts** : Bool – Includes conflicts information in response. Ignored if include_docs isn’t true. Default is false. Used for a filter query only.
* **descending** : Bool – Return the documents in descending by key order. Default is false. Used for a filter query only.
* **endkey** : String – Stop returning records when the specified key is reached. Optional. Used for a filter query only.
* **endkey_docid** : String – Stop returning records when the specified document ID is reached. Optional. Used for a filter query only.
* **include_docs** : Bool – Include the full content of the documents in the return. Default is false. Used for a filter query only.
* **inclusive_end** : Bool – Specifies whether the specified end key should be included in the result. Default is true. Used for a filter query only.
* **key** : String – Return only documents that match the specified key. Optional. Used for a filter query only.
* **keys** : [String] – Return only documents that match the specified keys. Optional. Used for a filter query only.
* **limit** : Int – Limit the number of the returned documents to the specified number. Optional.
* **skip** : Int – Skip this number of records before starting to return the results. Default is 0.
* **stale** : CouchDBQueryStale – Allow the results from a stale view to be used, without triggering a rebuild of all views within the encompassing design doc. Supported values: ok and update_after. Optional, default is .ignore. Used for a filter query only.
* **startkey** : String – Return records starting with the specified key. Optional. Used for a filter query only.
* **startkey_docid** : String – Return records starting with the specified document ID. Optional. Used for a filter query only.
* **update_seq** : Bool – Response includes an update_seq value indicating which sequence id of the underlying database the view reflects. Default is false. Used for a filter query only.
* **selector** : [String:Any] – JSON object describing criteria used to select documents. More information provided in the section on selector syntax. Used for a find query only.
* **sort** : [String:Any] – Name/value array following sort syntax. Optional. Used for a find query only.
* **fields** : [String:Any] – Name/value array specifying which fields of each object should be returned. If it is omitted, the entire object is returned. More information provided in the section on filtering fields. Optional. Used for a find query only.
* **use_index** : [String:Any] – Instruct a query to use a specific index. Specified either as "&lt;design\_document>" or ["&lt;design\_document>", "&lt;index\_name>"]. Optional. Used for a find query only.


## Error Responses

### enum CouchDBResponse

The CouchDBResponse enum details the available conditions encountered when executing a request to the CouchDB server.

* **.undefined** : Undefined.
* **.ok** : Request completed successfully.
* **.created** : Document created successfully.
* **.accepted** : Request has been accepted, but the corresponding operation may not have completed. This is used for background operations, such as database compaction.
* **.notModified** : The additional content requested has not been modified. This is used with the ETag system to identify the version of information returned.
* **.badRequest** : Bad request structure. The error can indicate an error with the request URL, path or headers. Differences in the supplied MD5 hash and content also trigger this error, as this may indicate message corruption.
* **.unauthorized** : The item requested was not available using the supplied authorization, or authorization was not supplied.
* **.forbidden** : The requested item or operation is forbidden.
* **.notFound** : The requested content could not be found. The content will include further information, as a JSON object, if available. The structure will contain two keys, error and reason. For example `{"error":"not_found","reason":"no_db_file"}`
* **.resourceNotAllowed** : A request was made using an invalid HTTP request type for the URL requested. For example, you have requested a PUT when a POST is required. Errors of this type can also triggered by invalid URL strings.
* **.notAcceptable** : The requested content type is not supported by the server.
* **.conflict** : Request resulted in an update conflict.
* **.preconditionFailed** : The request headers from the client and the capabilities of the server do not match.
* **.badContentType** : The content types supported, and the content type of the information being requested or submitted indicate that the content type is not supported.
* **.requestedRangeNotSatisfiable** : The range specified in the request header cannot be satisfied by the server.
* **.expectationFailed** : When sending documents in bulk, the bulk load operation failed.
* **.internalServerError** : The request was invalid, either because the supplied JSON was invalid, or invalid information was supplied as part of the request.
