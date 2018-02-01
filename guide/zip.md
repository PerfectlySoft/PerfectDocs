# Perfect's Zip Toolkit

Perfect provides a wrapper around the minizip C library and implements a set of convenience functions for use when the PerfectZip package is imported.

## Relevant Examples

* [Perfect-Zip-Example](https://github.com/PerfectExamples/Perfect-Zip-Example)

## Getting Started

On MacOS, install minizip using Homebrew:

```
brew install minizip
```

On Ubuntu, install minizip:

```
apt-get install libminizip-dev
```

In addition to the PerfectLib, you will need the Perfect-Zip dependency in the Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Zip.git", majorVersion: 3)
```

## Using Perfect Zip

The two main functions of the Perfect Zip module are to zip files, or unzip a zip file.

### Declaring an instance of the Zip class

Before initiating compression or decompression, an instance of the class must be created:

``` swift
let myVar = Zip()
```


### Zip

To compress a file, use the `.zipFiles(...)` method.

``` swift
zipFiles(
	paths: [String], 
	zipFilePath: String, 
	overwrite: Bool, 
	password: String?
) -> ZipStatus
```

This method returns the success/fail status of the operation as a `ZipStatus` enum.

#### Parameters:

* **paths:** The array of file paths to add to the zip file
* **zipFilePath:** The path and filename of the destination zip file
* **overwrite:** A boolean declaring the behaviour to attempt when the destination zip file exists
* **password:** The password string to use for password-protecting the zip file. Optional. Leave empty or omit to create an unprotected zip file.


### UnZip

To decompress a file, use the `.unzipFile(...)` method.

``` swift
unzipFile(
	source: String, 
	destination: String, 
	overwrite: Bool, 
	password: String = ""
) -> ZipStatus
```

This method returns the success/fail status of the operation as a `ZipStatus` enum.

#### Parameters:

* **source:** The file path of zipped file
* **destination:** The path to the directory into which the contents of the zip file are to be placed
* **overwrite:** A boolean declaring the behaviour to attempt when the destination directory or file exists
* **password:** The password string to use for decrypting a password-protected zip file. Optional.


### ZipStatus

The `ZipStatus` values are as follows:

* .FileNotFound
* .UnzipFail
* .ZipFail
* .ZipCannotOverwrite
* .ZipSuccess

The enum has a `.description` variable which returns human-readable descriptors of each enum value.

* .FileNotFound - **"File not found."**
* .UnzipFail - **"Failed to unzip file."**
* .ZipFail - **"Failed to zip file."**
* .ZipCannotOverwrite - **"Cannot overwrite destination file."**
* .ZipSuccess - **"Success."**



## Usage

The following will zip the specified directory:

``` swift
import PerfectZip

let myZip = Zip()

let thisZipFile = "/path/to/ZipFile.zip"
let sourceDir = "/path/to/files/"

let ZipResult = myZip.zipFiles(
    paths: [sourceDir], 
    zipFilePath: thisZipFile, 
    overwrite: true, password: ""
)
print("ZipResult Result: \(ZipResult.description)")
```

To unzip a file:

``` swift
import PerfectZip

let myZip = Zip()

let sourceDir = "/path/to/files/"
let thisZipFile = "/path/to/ZipFile.zip"

let UnZipResult = myZip.unzipFile(
    source: thisZipFile, 
    destination: sourceDir, 
    overwrite: true
)
print("Unzip Result: \(UnZipResult.description)")
```
