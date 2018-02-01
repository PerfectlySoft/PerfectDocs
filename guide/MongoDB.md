# MongoDB
The MongoDB Connector provides a Swift wrapper around the mongo-c client library, enabling access to MongoDB servers.

This package builds with the Swift Package Manager and is part of the
[Perfect](https://github.com/PerfectlySoft/Perfect) project. It was written to
be standalone, and does not require PerfectLib or any other components.

Ensure you have installed and activated the latest Swift 4.0 toolchain.

## Relevant Examples

* [MongoDBStORM-Demo](https://github.com/PerfectExamples/MongoDBStORM-Demo)
* [Perfect-Session-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Session-MongoDB-Demo)
* [Perfect-Turnstile-MongoDB-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MongoDB-Demo)


## Platform-Specific Preparation

### OS X

This package requires the [Homebrew](http://brew.sh) build of mongo-c.

To install Homebrew:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

To install mongo-c:

```
brew install mongo-c-driver
```

### Linux

Ensure that you have installed libmongoc.

```
sudo apt-get install libmongoc
```

### Including the MongoDB Driver in Your Project

Add this project as a dependency in your Package.swift file.

``` swift
.Package(
	url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", 
	majorVersion: 3
	)
```

For more information about using Perfect libraries with your project, see the chapter on "[Building with Swift Package Manager](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/buildingWithSPM.md)".

### Quick Start

The following will clone an empty starter project:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
```

Add this to the Package.swift file:

``` swift
let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url:"https://github.com/PerfectlySoft/Perfect.git", versions: Version(0,0,0)..<Version(10,0,0)),
		.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
	]
)
```

Create the Xcode project:

```
swift package generate-xcodeproj
```

Open the generated `PerfectTemplate.xcodeproj` file in Xcode.

The project will now build in Xcode and start a server on localhost port 8181.

>   **Important:** When a dependency has been added to the project, the Swift Package Manager must be invoked to generate a new Xcode project file. Be aware that any customizations that have been made to this file will be lost.

### Importing MongoDB for Use in Your Project

At the head of your Swift file, import the MongoDB package:

``` swift
import MongoDB
```

### Creating a MongoDB Connection

When you are opening a new connection to a MongoDB server, firstly obtain the connection URL. This will be the fully qualified domain name or IP address of the MongoDB server, with an optional port. 

Once you know the connection URL, open a connection as follows:

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
```

Where "localhost" is replaced by the actual server address.

### Defining a Database

Once the connection has been opened, a database can be assigned:

``` swift
let db = client.getDatabase(name: "test")
```

### Defining a MongoDB Collection

In order to work with a MongoDB Collection, it must be defined:

``` swift
let collection = db.getCollection(name: "testcollection")
```

### Closing Open Connections

Once a connection and associated connections are defined, it is wise to set them up to be closed using a ```defer``` statement.

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```
### Performing a Find

Use the ```find``` method to find all documents in the collection:

``` swift
    let fnd = collection.find(query: BSON())

    // Initialize empty array to receive formatted results
    var arr = [String]()

    // The "fnd" cursor is typed as MongoCursor, which is iterable
    for x in fnd! {
        arr.append(x.asString)
    }

```

For more detailed documentation on the MongoDB Collections class, see the chapter on [MongoDB Collections](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/MongoDB-Collections.md).
