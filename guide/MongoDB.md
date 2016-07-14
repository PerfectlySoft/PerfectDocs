# MongoDB
The MongoDB Connector provides a Swift wrapper around the mongo-c client library,
enabling access to MongoDB servers.

This package builds with Swift Package Manager and is part of the
[Perfect](https://github.com/PerfectlySoft/Perfect) project. It was written to
be stand-alone and so does not require PerfectLib or any other components.

Ensure you have installed and activated the latest Swift 3.0 tool chain.

## Platform specific preparation

### OS X

This package requires the [Home Brew](http://brew.sh) build of mongo-c.

To install Home Brew:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

To install mongo-c:

```
brew install mongo-c
```

### Linux

Ensure that you have installed libmongoc.

```
sudo apt-get install libmongoc
```

## Including the MongoDB Driver in your project

Add this project as a dependency in your Package.swift file.

```
.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
```

For more information about using Perfect libraries with your project please see the chapter on "Building with Swift Package Manager".

Quick Start
-----------

The following will clone an empty starter project:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
```

Add to the Package.swift file the dependency:

```
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

>   **Important:** When a dependancy has been added to the project, the Swift
>   Package Manager must be invoked to generate a new Xcode project file. Be
>   aware that any customizations that have been made to this file will be lost.

### Importing MongoDB for use in your project

At the head of your Swift file, import the MongoDB package:

``` swift
import MongoDB
```

### Creating a MongoDB connection

When you are opening a new connection to a MongoDB server, firstly obtain the connection URL. This will be the fully qualified domain name or IP address of the MongoDB server, with an optional port. 

Once you know the connection URL, open a connection as follows:

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
```

Where "localhost" is replaced by the actual server address.

### Defining a database

Once the connection has been opened, a database can be assigned:

``` swift
let db = client.getDatabase(name: "test")
```

### Defining a collection to work with

In order to work with a MongoDB Collection, it first be defined:

``` swift
let collection = db.getCollection(name: "testcollection")
```

### Closing open connections

Once a connection and associated connections are defined, it is wise to set them up to be closed using a ```defer``` statement.

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```
### Performing a Find

Using the ```find``` method to find all documents in the collection:

``` swift
    let fnd = collection.find(query: BSON())

    // Initialize empty array to receive formatted results
    var arr = [String]()

    // The "fnd" cursor is typed as MongoCursor, which is iterable
    for x in fnd! {
        arr.append(x.asString)
    }

```

For more detailed documentation on the MongoCollection class, please see the MongoCollection documentation.