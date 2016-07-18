# File Operations
Perfect brings file system operations into your sever side Swift environment in an accessible, common sense fashion.

First, ensure the `PerfectLib` is imported in your Swift file:

``` swift
import Perfectib
```
You are now able to use the `File` object to query and manipulate the file system.

### Setting up a File object reference

Specify the absolute or relative path to the file:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
```


### Opening a file for read or write access

> **Important:** Before writing to a file - even if it is a new file - the file must be opened with the appropriate permissions.

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

For full outlines of [OpenMode](#OpenMode) and [PermissionMode](PermissionMode) values please see their definitions later in this document.

### Checking if a file exists

Use the `exists` method to return a boolean value.

``` swift
thisFile.exists
```


### Get the modification time for a file

Return the modification date for the file in the standard UNIX format of seconds since 1970/01/01 00:00:00 GMT, as an Integer.

``` swift
thisFile.modificationTime
```

### File paths

Regardless of how a file reference was defined, both the absolute (real) and internal path can be returned.

Return the "internal reference" file path:

``` swift
thisFile.path
```

Return the "real" file path. If the file is a symbolic link, the link will be resolved:

``` swift
thisFile.realPath
```

### Closing a file

Once a file has been opened for read or write, it is advisable to close either at a specific place in code, or using `defer`

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
// Your processing here
thisFile.close()
```


### Deleting a file

To remove a file from the file system, use the `delete()` method. 

``` swift
thisFile.delete()
```

This also closes the file, so there is no need to invoke an additional `close()` method.



### Returning the size of a file

`size` returns the size of the file in bytes, as an integer.

``` swift
thisFile.size
```

### Determining if the file is a symbolic link

If the file is a symbolic link, the method will return boolean `true`, otherwise `false`.

``` swift
thisFile.isLink
```

### Determining if the file object is a directory

If the file object refers instead to a directory, `isDir` will return boolean `true`, otherwise `false`.

``` swift
thisFile.isDir
```

### Returning the file's UNIX permissions

`perms` returns the UNIX style permissions for the file as a `PermissionMode` object.

``` swift
thisFile.perms
```

For example:

``` swift
print(thisFile.perms)
>> PermissionMode(rawValue: 29092)
```

## Reading contents of a file

### readSomeBytes

Reads up to the indicated number of bytes from the file:

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: <Int>)
```

#### Parameters

* **count:** The maximum number of bytes to read

For example, to read the first 10 bytes of a file:


``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readSomeBytes(count: 10)
print(contents)
>> [35, 32, 80, 101, 114, 102, 101, 99, 116, 84]
```



### readString

`readString` reads the entire file as a string

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let contents = try thisFile.readString()
```

## Writing, Copying and Moving Files

> **Important:** Before writing to a file - even if it is a new file - the file must be opened with the appropriate permissions.


### Writing a string to a file

Use `write` to create or rewrite a string to the file using UTF-8 encoding. The method returns an integer which is the number of bytes written.

Note that this method uses the `@discardableResult` property so can be used without assignment if required.

``` swift
let bytesWritten = try thisFile.write(string: <String>)
```

### Writing a bytes array to a file

An array of `bytes` can also be written directly to a file. The method returns an integer which is the number of bytes written.

Note that this method uses the `@discardableResult` property so can be used without assignment if required.

``` swift
let bytesWritten = try thisFile.write(
	bytes: <[UInt8]>, 
	dataPosition: <Int>, 
	length: <Int>
	)
```

#### Parameters

* **bytes:** The array of UInt8 to write
* **dataPosition:** *Optional.* The offset within `bytes` at which to begin writing.
* **length:** *Optional.* The number of bytes to write.



### Moving a file

Once a file is defined, the `moveto` method can be used to relocate the file to a new location in the file system. This can also be used to rename a file if desired. The operation returns a new file object representing the new location.

``` swift
let newFile = thisFile.moveTo(path: <String>, overWrite: <Bool>)
```

#### Parameters

* **path:** The path to move the file to
* **overWrite:** *Optional.* Indicates that any existing file at the destination path should first be deleted. Default is `false`

#### Error handling

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.moveTo(path: "/path/to/file/goodbyeWorld.txt")
```



### Copying a file

Similar to `moveTo`, `copyTo` copies the file to the new location, optionally overwriting any existing file - however it does not delete the original file. A new file object is returned representing the new location.

Note that this method uses the `@discardableResult` property so can be used without assignment if required.

``` swift
let newFile = thisFile.copyTo(path: <String>, overWrite: <Bool>)
```
#### Parameters

* **path:** The path to copy the file to
* **overWrite:** *Optional.* Indicates that any existing file at the destination path should first be deleted. Default is `false`

#### Error handling

The method throws `PerfectError.FileError` on error.

``` swift
let thisFile = File("/path/to/file/helloWorld.txt")
let newFile = try thisFile.copyTo(path: "/path/to/file/goodbyeWorld.txt")
```


## File Locking Functions

The file locking functions allow *sections* of a file to be locked with *advisory-mode locks.* 

All the locks for a file are removed when the file is closed or the process terminates.

> **Note:** These are not file system locks, and do not prevent others from performing write operations on the affected files: The locks are "advisory-mode locks".

### Locking a file

Attempts to place an advisory lock starting from the current position marker up to the indicated byte count. This function will block the current thread until the lock can be performed.

``` swift
let result = try thisFile.lock(byteCount: <Int>)
```

### Unlocking a file

Unlocks the number of bytes starting from the current position marker up to the indicated byte count.

``` swift
let result = try thisFile.unlock(byteCount: <Int>)
```

### Attempt to lock a file

Attempts to place an advisory lock starting from the current position marker up to the indicated byte count. This function will throw an exception if the file is already locked, but will not block the current thread.

``` swift
let result = try thisFile.tryLock(byteCount: <Int>)
```


### Testing a lock

Tests if the indicated bytes are locked. Returns a boolean true or false.

``` swift
let isLocked = try thisFile.testLock(byteCount: <Int>)
```

## File OpenMode

The OpenMode of a file is defined as an enum:

* .read: Opens the file for read-only access.
* .write: Opens the file for write-only access, creating the file if it did not exist.
* .readWrite: Opens the file for read-write access, creating the file if it did not exist.
* .append: Opens the file for read-write access, creating the file if it did not exist and moving the file marker to the end.
* .truncate: Opens the file for read-write access, creating the file if it did not exist and setting the file's size to zero.

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

### PermissionMode options

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


