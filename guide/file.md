# File Operations
Perfect brings file system operations into your sever-side Swift environment to control how data is stored and retrieved in an accessible way.

First, ensure the `PerfectLib` is imported in your Swift file:

``` swift
import PerfectLib
```
You are now able to use the `File` object to query and manipulate the file system.

### Setting Up a File Object Reference

Specify the absolute or relative path to the file:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
```
If you are not familiar with the file path, please read [Directory Operations](https://github.com/PerfectlySoft/PerfectDocs/edit/master/guide/dir.md) first.

### Opening a File for Read or Write Access

> **Important:** Before writing to a file — even if it is a new file — it must be opened with the appropriate permissions.

To open a file:

``` swift
try thisFile.open(<OpenMode>,permissions:<PermissionMode>)
```

For example, to write a file:

``` swift
let thisFile = File("helloWorld.txt")
try thisFile.open(.readWrite)
try thisFile.write(string: "Hello, World!")
thisFile.close()
```

For full outlines of [OpenMode](#OpenMode) and [PermissionMode](PermissionMode) values, see their definitions later in this document.

### Checking If a File Exists

Use the `exists` method to return a Boolean value.

``` swift
thisFile.exists
```


### Get the Modification Time for a File

Return the modification date for the file in the standard UNIX format of seconds since 1970/01/01 00:00:00 GMT, as an integer using:

``` swift
thisFile.modificationTime
```

### File Paths

Regardless of how a file reference was defined, both the absolute (real) and internal path can be returned.

Return the "internal reference" file path:

``` swift
thisFile.path
```

Return the "real" file path. If the file is a symbolic link, the link will be resolved:

``` swift
thisFile.realPath
```

### Closing a File

Once a file has been opened for read or write, it is advisable to either close it at a specific place within the code, or by using `defer`:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
// Your processing here
thisFile.close()
```


### Deleting a File

To remove a file from the file system, use the `delete()` method. 

``` swift
thisFile.delete()
```

This also closes the file, so there is no need to invoke an additional `close()` method.



### Returning the Size of a File

`size` returns the size of the file in bytes, as an integer.

``` swift
thisFile.size
```

### Determining if the File is a Symbolic Link

If the file is a symbolic link, the method will return Boolean `true`, otherwise `false`.

``` swift
thisFile.isLink
```

### Determining if the File Object is a Directory

If the file object refers instead to a directory, `isDir` will return either a Boolean `true` or `false` value.

``` swift
thisFile.isDir
```

### Returning the File's UNIX Permissions

`perms` returns the UNIX style permissions for the file as a `PermissionMode` object.

``` swift
thisFile.perms
```

For example:

``` swift
print(thisFile.perms)
>> PermissionMode(rawValue: 29092)
```

## Reading Contents of a File

### readSomeBytes

Reads up to the indicated number of bytes from the file:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: <Int>)
```

#### Parameters

To read a specific byte range of a file's contents, enter the number of bytes you wish to read. For example, to read the first 10 bytes of a file:


``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: 10)
print(contents)
>> [35, 32, 80, 101, 114, 102, 101, 99, 116, 84]
```



### readString

`readString` reads the entire file as a string:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readString()
```

## Writing, Copying, and Moving Files

> **Important:** Before writing to a file — even if it is a new file — it must be opened with the appropriate permissions.


### Writing a String to a File

Use `write` to create or rewrite a string to the file using UTF-8 encoding. The method returns an integer which is the number of bytes written.

Note that this method uses the `@discardableResult` property, so it can be used without assignment if required.

``` swift
let bytesWritten = try thisFile.write(string: <String>)
```

### Writing a Bytes Array to a File

An array of `bytes` can also be written directly to a file. The method returns an integer which is the number of bytes written.

Note that this method uses the `@discardableResult` property, so it can be used without assignment if required.

``` swift
let bytesWritten = try thisFile.write(
	bytes: <[UInt8]>, 
	dataPosition: <Int>, 
	length: <Int>
	)
```

#### Parameters

* **bytes:** The array of UInt8 to write
* **dataPosition:** *Optional.* The offset within `bytes` at which to begin writing
* **length:** *Optional.* The number of bytes to write


### Moving a File

Once a file is defined, the `moveto` method can be used to relocate the file to a new location in the file system. This can also be used to rename a file if desired. The operation returns a new file object representing the new location.

``` swift
let newFile = thisFile.moveTo(path: <String>, overWrite: <Bool>)
```

#### Parameters

* **path:** The path to move the file to
* **overWrite:** *Optional.* Indicates that any existing file at the destination path should first be deleted. Default is `false`

#### Error Handling

The method throws `PerfectError.FileError` on error.

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.moveTo(path: "/path/to/file/goodbyeWorld.txt")
```

### Copying a File

Similar to `moveTo`, `copyTo` copies the file to the new location, optionally overwriting any existing file. However, it does not delete the original file. A new file object is returned representing the new location.

Note that this method uses the `@discardableResult` property, so it can be used without assignment if required.

``` swift
let newFile = thisFile.copyTo(path: <String>, overWrite: <Bool>)
```
#### Parameters

* **path:** The path to copy the file to
* **overWrite:** *Optional.* Indicates that any existing file at the destination path should first be deleted. Default is `false`

#### Error Handling

The method throws `PerfectError.FileError` on error.

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.copyTo(path: "/path/to/file/goodbyeWorld.txt")
```

### File Locking Functions

The file locking functions allow *sections* of a file to be locked with *advisory-mode locks.* 

All the locks for a file are removed when the file is closed or the process terminates.

> **Note:** These are not file system locks, and do not prevent others from performing write operations on the affected files: The locks are "advisory-mode locks".

### Locking a File

Attempts to place an advisory lock starting from the current position marker up to the indicated byte count. This function will block the current thread until the lock can be performed.

``` swift
let result = try thisFile.lock(byteCount: <Int>)
```

### Unlocking a File

Unlocks the number of bytes starting from the current position marker up to the indicated byte count.

``` swift
let result = try thisFile.unlock(byteCount: <Int>)
```

### Attempt to Lock a File

Attempts to place an advisory lock starting from the current position marker up to the indicated byte count. This function will throw an exception if the file is already locked, but will not block the current thread.

``` swift
let result = try thisFile.tryLock(byteCount: <Int>)
```

### Testing a Lock

Tests if the indicated bytes are locked. Returns a Boolean true or false.

``` swift
let isLocked = try thisFile.testLock(byteCount: <Int>)
```

## File OpenMode

The OpenMode of a file is defined as an enum:

* .read: Opens the file for read-only access
* .write: Opens the file for write-only access, creating the file if it did not exist
* .readWrite: Opens the file for read-write access, creating the file if it did not exist
* .append: Opens the file for read-write access, creating the file if it did not exist and moving the file marker to the end
* .truncate: Opens the file for read-write access, creating the file if it did not exist and setting the file's size to zero

For example, to write a file:

``` swift
let thisFile = File("helloWorld.txt")
try thisFile.open(.readWrite)
try thisFile.write(string: "Hello, World!")
thisFile.close()
```

## File PermissionMode

The PermissionMode for a directory or file is provided as a single option or as an array of options.

For example, to create a directory with specific permissions:

``` swift
let thisDir = Dir("/path/to/dir/")
do {
	try thisDir.create(perms: [.rwxUser, .rxGroup, .rxOther])
} catch {
	print("error")
}
//or
do {
	try thisDir.create(perms: .rwxUser)
} catch {
	print("error")
}
```

### PermissionMode Options

* `.readUser`: Readable by user
* `.writeUser`: Writable by user
* `.executeUser`: Executable by user
* `.readGroup`: Readable by group
* `.writeGroup`: Writable by group
* `.executeGroup`: Executable by group
* `.readOther`: Readable by others
* `.writeOther`: Writable by others
* `.executeOther`: Executable by others
* `.rwxUser`: Read, write, execute by user
* `.rwUserGroup`: Read, write by user and group
* `.rxGroup`: Read, execute by group
* `.rxOther`: Read, execute by other


