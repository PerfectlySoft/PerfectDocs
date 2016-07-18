# Directory Operations

Perfect brings file system operations into your sever side Swift environment in an accessible, common sense fashion.

First, ensure the `PerfectLib` is imported in your Swift file:

``` swift
import Perfectib
```
You are now able to use the `Dir` object to query and manipulate the file system.

### Setting up a Directory object reference

Specify the absolute or relative path to the directory:

``` swift
let thisDir = Dir("/path/to/directory/")
```

### Checking if a directory exists

Use the `exists` method to return a boolean value.

``` swift
let thisDir = Dir("/path/to/directory/")
thisDir.exists
```

### Returning the current directory object's name

`name` returns the name of the object's directory. Note that this is different from the "path".

``` swift
thisDir.name
```

### Returning the parent directory

`parentDir` returns a `Dir` object representing the current directory object's parent. Returns nil if there is no parent.

``` swift
let thisDir = Dir("/path/to/directory/")
let parent = thisDir.parentDir
```

### Revealing the directory path

`path` returns the path to the current directory.

``` swift
let thisDir = Dir("/path/to/directory/")
let path = thisDir.path
```

### Returning the directory's UNIX permissions

`perms` returns the UNIX style permissions for the directory as a `PermissionMode` object.

``` swift
thisDir.perms
```

For example:

``` swift
print(thisDir.perms)
>> PermissionMode(rawValue: 29092)
```

### Creating a directory

Creates the directory using the provided permissions. All directories along the path will be created if needed.

The following will create a new directory with the default permissions (Owner: read-write-execute, Group and Everyone: read-exexute.

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


### Deleting a directory

Deleting a directory from the file system:

``` swift
let newDir = Dir("/path/to/directory/newDirectory")
try newDir.delete()
```

The method throws `PerfectError.FileError` if an error deleting the directory was encountered.




## Working Directories

### Set the working directory to the location of the current object

Use `setAsWorkingDir` to set the current working directory to the location of the object's path.

``` swift
let thisDir = Dir("/path/to/directory/")
try thisDir.setAsWorkingDir()
```


### Return the current working directory

Returns a new object containing the current working directory.

``` swift
let workingDir = Dir.workingDir
```


## Reading the directory structure

`forEachEntry` enumerates the contents of the directory passing the name of each contained element to the provided callback.

``` swift 
try thisDir.forEachEntry(closure: {
	n in
	print(n)
})
```
