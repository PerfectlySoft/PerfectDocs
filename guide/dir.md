# Directory Operations

Perfect brings file system operations into your sever-side Swift environments to control how data is stored and retrieved in an accessible way.

## Relevant Examples

* [Perfect-Directory-Lister](https://github.com/PerfectExamples/Perfect-Directory-Lister)
* [Perfect-FileHandling](https://github.com/PerfectExamples/Perfect-FileHandling)

## Usage

First, ensure the `PerfectLib` is imported in your Swift file:

``` swift
import PerfectLib
```
You are now able to use the `Dir` object to query and manipulate the file system.

### Setting Up a Directory Object Reference

Specify the absolute or relative path to the directory:

``` swift
let thisDir = Dir("/path/to/directory/")
```

### Checking If a Directory Exists

Use the `exists` method to return a Boolean value.

``` swift
let thisDir = Dir("/path/to/directory/")
thisDir.exists
```

### Returning the Current Directory Object's Name

`name` returns the name of the object's directory. Note that this is different from the "path".

``` swift
thisDir.name
```

### Returning the Parent Directory

`parentDir` returns a `Dir` object representing the current directory object's parent. Returns nil if there is no parent.

``` swift
let thisDir = Dir("/path/to/directory/")
let parent = thisDir.parentDir
```

### Revealing the Directory Path

`path` returns the path to the current directory.

``` swift
let thisDir = Dir("/path/to/directory/")
let path = thisDir.path
```

### Returning the Directory's UNIX Permissions

`perms` returns the UNIX style permissions for the directory as a `PermissionMode` object.

``` swift
thisDir.perms
```

For example:

``` swift
print(thisDir.perms)
>> PermissionMode(rawValue: 29092)
```

### Creating a Directory

`create` creates the directory using the provided permissions. All directories along the path will be created if needed.

The following will create a new directory with the default permissions (Owner: read-write-execute, Group and Everyone: read-execute.

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.create()
```

To create a directory with specific permissions, specify the `perms` parameter:

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.create(perms: [.rwxUser, .rxGroup, .rxOther])
```

The method throws `PerfectError.FileError` if an error creating the directory was encountered.


### Deleting a Directory

Deleting a directory from the file system:

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.delete()
```

The method throws `PerfectError.FileError` if an error deleting the directory was encountered.

### Working Directories

### Set the Working Directory to the Location of the Current Object

Use `setAsWorkingDir` to set the current working directory to the location of the object's path.

``` swift
let thisDir = Dir("/path/to/directory/")
try thisDir.setAsWorkingDir()
```

### Return the Current Working Directory

Returns a new object containing the current working directory.

``` swift
let workingDir = Dir.workingDir
```

### Reading the Directory Structure

`forEachEntry` enumerates the contents of the directory, passing the name of each contained element to the provided callback.

``` swift 
try thisDir.forEachEntry(closure: {
	n in
	print(n)
})
```
