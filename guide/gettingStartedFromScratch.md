# Getting Started From Scratch

This guide will take you through the steps of settings up a simple HTTP server from scratch with Swift and Perfect.

## Prerequisites

### Swift 3.0

After you have installed a Swift 3.0 toolchain from [Swift.org](https://swift.org/getting-started/), open up a terminal window and type
```
swift --version
```

It will produce a message similar to this one:

```
Apple Swift version 3.0.1 (swiftlang-800.0.58.6 clang-800.0.42.1)
Target: x86_64-apple-macosx10.9
```
Make sure you are running the release version of Swift 3.0.1. Perfect will not compile successfully if you are running a version of Swift that is lower than 3.0.1.

You can find out which version of Swift you will need by looking in [the README of the main Perfect repo](https://github.com/PerfectlySoft/Perfect#compatibility-with-swift).

### OS X

Everything you need is already installed.

### Ubuntu Linux

Perfect runs in Ubuntu Linux 14.04, 15.10 and 16.04 environments. Perfect relies on OpenSSL, libssl-dev, and uuid-dev. To install these, in the terminal, type:

```
sudo apt-get install openssl libssl-dev uuid-dev
```

## Getting Started with Perfect

Now that you’re ready to build a simple web application from scratch, then go ahead and create a new folder where you will keep your project files:

```
mkdir MyAwesomeProject
cd MyAwesomeProject
```

As a good developer practice, make this folder a git repo:

```
git init
touch README.md
git add README.md
git commit -m "Initial commit"
```

It's also recommended to add a `.gitignore` similar to the contents of [this Swift .gitignore template from gitignore.io](https://www.gitignore.io/api/swift).

### Create the Swift Package

Now create a `Package.swift` file in the root of the repo with the following content. This is needed for the Swift Package Manager (SPM) to build the project.

``` swift
import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        .Package(
        url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git",
        majorVersion: 2, minor: 0
        )
    ]
)
```

Next create a folder called `Sources` and create a `main.swift` in there:

```
mkdir Sources
echo 'print("Well hi there!")' >> Sources/main.swift
```

Now the project is ready to be built and run by running by the following two commands:

```
swift build
.build/debug/MyAwesomeProject
```

You should see the following output:

```
Well hi there!
```

### Setting up the server

Now that the Swift package is up and running, the next step is to implement the Perfect-HTTPServer. Open up the `Sources/main.swift` and change its content the following:

``` swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

// Create HTTP server.
let server = HTTPServer()

// Register your own routes and handlers
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
		request, response in
		response.setHeader(.contentType, value: "text/html")
		response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
		response.completed()
	}
)

// Add the routes to the server.
server.addRoutes(routes)

// Set a listen port of 8181
server.serverPort = 8181

do {
	// Launch the HTTP server.
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("Network error thrown: \(err) \(msg)")
}
```

Build and run the project again with:

```
swift build
.build/debug/MyAwesomeProject
```

The server is now running and waiting for connections. Access [http://localhost:8181/](http://127.0.0.1:8181/) to see the greeting. Hit "control-c" to terminate the server.

### Xcode

Swift Package Manager (SPM) can generate an Xcode project which can run the PerfectTemplate server and provide full source code editing and debugging for your project. Enter the following in your terminal:

```
swift package generate-xcodeproj
```

Open the generated file "PerfectTemplate.xcodeproj" and add the following to the "Library Search Paths" for the project (not just the target):

```
$(PROJECT_DIR) - Recursive
```

Ensure that you have selected the executable target and selected it to run on "My Mac". Also ensure that the correct Swift toolchain is selected. You can now run and debug the server directly in Xcode.
