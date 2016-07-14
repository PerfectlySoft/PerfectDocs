# MongoDB
The MongoDB Connector provides a Swift wrapper around the mongo-c client library,
enabling access to MongoDB servers.

This package builds with Swift Package Manager and is part of the
[Perfect](https://github.com/PerfectlySoft/Perfect) project. It was written to
be stand-alone and so does not require PerfectLib or any other components.

Ensure you have installed and activated the latest Swift 3.0 tool chain.

OS X Build Notes
----------------

This package requires the [Home Brew](http://brew.sh) build of mongo-c.

To install Home Brew:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

To install mongo-c:

```
brew install mongo-c
```

Linux Build Notes
-----------------

Ensure that you have installed libmongoc.

Building
--------

Add this project as a dependency in your Package.swift file.

```
.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", versions: Version(0,0,0)..<Version(10,0,0))
```

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

Creating a MongoDB connection and query a collection
----------------------------------------------------

In Xcode, open Sources/PerfectTemplate/main.swift, and update the code to the
following:

```
import PerfectLib

// Initialize base-level services
PerfectServer.initializeServices()

// Add routes
addURLRoutes()

do {
    // Launch the HTTP server on port 8181
    try HTTPServer(documentRoot: "./webroot").start(port: 8181)
} catch PerfectError.networkError(let err, let msg) {
    print("Network error thrown: \(err) \(msg)")
}
```

Create a new file at the same level as `main.swift`, called
`routingHandlers.swift`

Next

-   add the import directives for PerfectLib and MongoDB connector;

-   add some testing routes;

-   register the routes with Perfect.

```
import PerfectLib
import MongoDB

func addURLRoutes() {
    Routing.Routes["/test" ] = testHandler
    Routing.Routes["/mongo" ] = mongoHandler
}

// Register any handlers or perform any one-time tasks.
public func PerfectServerModuleInit() {
    addURLRoutes()
}
```

Note that the two routes are handed off to functions.

The function handling the “/test” url simply return some “Hello, World!” JSON.

```
func testHandler(request: WebRequest, _ response: WebResponse) {
    let returning = "{Hello, World!}"
    response.appendBody(string: returning)
    response.requestCompleted()
}
```

More information is available in the “URL Routing” example
(<https://github.com/PerfectlySoft/PerfectExample-URLRouting>)

A sample MongoDB handler function would be as follows:

```
func mongoHandler(request: WebRequest, _ response: WebResponse) {

    // open a connection
    let client = try! MongoClient(uri: "mongodb://localhost")

    // set database, assuming "test" exists
    let db = client.getDatabase(name: "test")

    // define collection
    guard let collection = db.getCollection(name: "testcollection") else {
        return
    }

    // Here we clean up our connection, 
    // by backing out in reverse order created
    defer {
        collection.close()
        db.close()
        client.close()
    }

    // Perform a "find" on the perviously defined collection
    let fnd = collection.find(query: BSON())

    // Initialize empty array to receive formatted results
    var arr = [String]()

    // The "fnd" cursor is typed as MongoCursor, which is iterable
    for x in fnd! {
        arr.append(x.asString)
    }

    // return a formatted JSON array.
    let returning = "{\"data\":[\(arr.joined(separator: ","))]}"

    // Return the JSON string
    response.appendBody(string: returning)
    response.requestCompleted()
}
```