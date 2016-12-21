# MongoDB GridFS

GridFS is a specification for storing and retrieving files that exceed the BSON-document size limit of 16 MB by dividing a single document into parts. Perfect-GridFS provides two different classes to access MongoDB GridFS:

- GridFS: list(), search(), delete(), upload(), download() and drop()
- GridFile: provides file info, download() and dedicated large file operations such as tell() / seek(), partially read()/write(), and download()

## GridFS Class

Accessing a GridFS object in a Perfect MongoDB context is simple. First, make sure that a mongo client is already available, then call `gridFS()` method with database name and a prefix string as an option. Check the example code below:

``` swift
// suppose let client = try MongoClient(uri: "mongodb://...")
let gridfs = try client.gridfs(database: "test")
```
or get the gripfs handle with an optional prefix:

``` swift
let gridfs = try client.gridfs(database: "test", prefix: "clip")
```


### GridFS Methods

GridFS has methods listed below:

- list(optional filter): show all / parts of files on the server
- drop(): drop all files and the gridfs system from the database
- search(filename): search a file on the server
- delete(filename): delete a file from the server
- upload(from, to): upload a local file to server
- download(from, to): download a remote to local path
- close(): close gridfs handle

#### List files

`list()` returns an array of GridFile objects with an optional BSON type parameter called `filter`, which can narrow down the list result by setting up a proper query. 

``` swift
func list(filter: BSON? = nil) throws -> [GridFile]
```

By default, `list()` doesn't require any parameters but actually will perform a listing in alphabetic order by setting a bson query with a key `$order` internally:

``` swift
let list = try gridfs.list()
print("\(list.count) file(s) found")
```

#### Drop the Current GridFS

Method `drop()` Requests that an entire GridFS be dropped, including all files associated with it.

``` swift
gridfs.drop()
```


#### Search for a file

To search for a file on server and get information or content back, use `search()` method. It will return a `GridFile` object which will be discussed later in this article. If failed, it will throw a MongoClientError:

``` swift
let f = try gridfs.search(name: "file.name.on.server")
print(f.id)
print(f.length)
```

#### Delete a file
Call method `delete()` to remove a file on server. If failed, it will throw a MongoClientError:

``` swift
try gridfs.delete(name: "file.name.on.server")
```

#### Upload a file

Specify a local file path and its expected remote file name to upload, then `upload()` will call back with the uploaded gridFile handle:

``` swift
gridfs.upload(from: "/path/to/local.file", to: "name.on.server.type") { gridFile in 
	guard gridFile != nil else {
		// throw some error here
	}//end guard
	// print out the uploaded file length
	print(gridFile?.length ?? 0)
}//end upload
```

The example above is highly recommended for large file operations which usually takes some time to complete, however, there is also a synchronized way of upload which could block the current thread, which could be possibly a convenient way for small files:

``` swift
let gridFile = gridfs.upload(from: "/path/to/local", to: "name.on.server")
if gridFile != nil {
	print("Upload Succeed");
}//end if
```

#### Download a file

Specify a local path to download a file existing on server then call back once downloaded, and throws a MongoClientError if failed:

``` swift
do {
	try gridfs.download(from: "name.on.server", to: "/local/path/downloaded.name") { bytes in 
		print("Totally \(bytes) downloaded.")
	}//end download
}catch(let err) {
	print("Unexpected \(err) in downloading")
}
```

Similar to `upload()`, `download()` also has an alternative blocking interface without completion callback, which is not suggested for large files:

``` swift
do {
	let bytes = try gridfs.download(from: "name.on.server", to: "/local/path/downloaded.name") 
	print("Totally \(bytes) downloaded.")
}catch(let err) {
	print("Unexpected \(err) in downloading")
}
```

#### Closing the Connection

Practically, swift GC mechanism will automatically release the `GridFS` / `GridFile` objects and related resources, however, users can also call defer closing manually if preferred:

``` swift
defer {
	gridfs.close()
}//end defer
```

## GridFile class

Method `search()` and `list()` return `GridFile` and `[GridFile]` array correspondingly. GridFile is for dealing with file information and more complicated file operations such as file cursor and partially read / write, as described below:

### GridFile properties:

- id: oid string property of GridFile.
- md5: md5 string property of GridFile. 
- aliases: a BSON property which represents the aliases of the GridFile.
- contentType: string property of GridFile. 
- length: length of the GridFile, an Int64 number. Read only
- uploadDate: Int64 unix epoch time stamp of the GridFile. Read only
- fileName: file name stored in the remote server. 
- metaData: a BSON type property of GridFile to hold the meta data. 

### GridFile methods:

Besides above properties, `GridFile` also has a few available methods as below:

- download(): directly download the whole file
- tell(): a UInt64 position of current gridfile instance's cursor is pointing at
- seek(cursor, optional whence): move the file cursor to objective position referring to whence it goes
- paritallyRead(amount, optional timeout): read a part of the file from the current file cursor position and return it in a binary buffer [UInt8]
- partiallyWrite(bytes, optional timeout): write a buffer to the current file cursor position
- close(): close the GridFile instance handle

#### download()

Method `download()` of GridFile class is the lower base of `GridFS.download()`, which actually calls `GridFile.download()` internally. If there is an available GridFile handler, simply apply the local path for downloading:

``` swift
do {
	try gridfile.download(to: "/local/path/downloaded.name") { bytes in 
		print("Totally \(bytes) downloaded.")
	}//end download
}catch(let err) {
	print("Unexpected \(err) in downloading")
}
```

Or, alternatively, download the file without threading:

``` swift
do {
	let bytes = try gridfile.download(to: "/local/path/downloaded.name") 
	print("Totally \(bytes) downloaded.")
}catch(let err) {
	print("Unexpected \(err) in downloading")
}
```

#### tell()

Method `tell()` will return a `UInt64` integer to represent the current cursor position of `GridFile` instance:

```
let position = gridfile.tell()
print(“current position is \(position)”)
```

#### seek()

Use `seek()` to move the current GridFile cursor to a new position, with an optional supporting parameter to indication whence the cursor will offset to, i.e., `.begin` as default for calculate offset from beginning of the file, `.current` for an offset from current position, and `.end` represents an offset from the every end of `GridFile`:

``` swift
 enum Whence {
 	 // offset from starting point of file
    case begin
    // offset from current file cursor
    case current
    // offset from the last byte of the file
    case end
 }//end whence
  
func seek(cursor: Int64, whence:Whence = .begin) throws
```

For example, the following code demonstrates how to move the file cursor to 1MB from the starting point:

``` swift
try gridfile.seek(1048576)
```

#### partiallyRead()

To deal with large files, GridFile provides partially read() / write() methods:

``` swift
func partiallyRead(amount: UInt32, timeout:UInt32 = 0) throws -> [UInt8] 
```

Parameter `amount` is the bytes count to read, and the `timeout` value represents the milliseconds to wait for reading. If failed to read, it would throw a `MongoClientError`. Please ⚠️*NOTE*⚠️ the reading starting point is controlled by `seek()` and is subjected to changed after every `paritallyRead()` or `partiallyWrite()`. The following example demonstrates how it works:

``` swift
let file = try gridfs.search("a.large.file")
let cursor = file?.tell()
print("if no operations before, the cursor should be zero, now it is \(cursor)")
let oneMegaByte = 1048576
// move the cursor to 100MB position from the file starting point
try file?.seek(cursor: Int64(oneMegaByte * 100))
// then read 1MB bytes into a [UInt8] buffer
let bytes = try file?.partiallyRead(amount: UInt32(oneMegaByte))
```

#### partiallyWrite()

Method `partiallyWrite()` performs an opposite operation against to `partiallyRead()`. If failed to write, it would throw a `MongoClientError`. :

``` swift
public func partiallyWrite(bytes:[UInt8], timeout:UInt32 = 0) throws -> Int
```

The first parameter `bytes` is the buffer to write, with a second optional parameter of timeout in milliseconds to wait for writing. If ignored, the writing operation would immediately return. ⚠️*Again*⚠️, the writing starting point is controlled by `seek()` and is subjected to changed after every `paritallyRead()` or `partiallyWrite()`:

``` swift
// move the file cursor to an offset of 1k from end of file
try file?.seek(cursor: 1024, whence: .end)
let buffer:[UInt8] = [1,2,3,4,5]
// write the array to the current position
let totalWritten = try file?.partially(bytes: buffer)
print(totalWritten)
```
If succeed, the above code would write five different bytes at the 1k point from end of the file.

#### close()

Practically, swift GC mechanism will automatically release the `GridFS` / `GridFile` objects and related resources, however, users can also call defer closing manually if preferred:

``` swift
defer {
	gridfile.close()
}//end defer
```
