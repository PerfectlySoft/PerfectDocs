# Getting Started From Scratch

This guide will take you through the steps of settings up a simple HTTP server from scratch with Swift and Perfect.

## Prerequisites

### Swift 4.0

After you have installed a Swift 4.0+ toolchain from [Swift.org](https://swift.org/getting-started/), open up a terminal window and type
```
swift --version
```

It will produce a message similar to this one:

```
Apple Swift version 4.0 (swiftlang-900.0.65 clang-900.0.37)
Target: x86_64-apple-macosx10.9
```
Make sure you are running the release version of Swift 4. Perfect will not compile successfully if you are running a version of Swift that is lower than 3.0.1.

You can find out which version of Swift you will need by looking in [the README of the main Perfect repo](https://github.com/PerfectlySoft/Perfect#compatibility-with-swift).

### macOS

Everything you need is already installed.

### Ubuntu Linux

Perfect runs in Ubuntu Linux 16.04 environments. Perfect relies on OpenSSL, libssl-dev, and uuid-dev. To install these, in the terminal, type:

```
sudo apt-get install openssl libssl-dev uuid-dev
```

## Getting Started with Perfect

Now that you’re ready to build a simple web application from scratch, then go ahead and create a new folder where you will keep your project files:

```
mkdir MyAwesomeProject
cd MyAwesomeProject
```

Initialize a new SPM package with:

```
swift package init --type executable
```

This will create a variety of files and will produce the following output:

```
Creating executable package: MyAwesomeProject
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/MyAwesomeProject/main.swift
Creating Tests/
```

### Create the Swift Package

Open the `Package.swift` file. You will see it has this content:

```
// swift-tools-version:4.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
            name: "MyAwesomeProject",
            dependencies: []),
    ]
)
```

Modify this file by adding a Perfect-HTTPServer dependency. The result should look like so:

```
// swift-tools-version:4.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyAwesomeProject",
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        .package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", from: "3.0.0")
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages which this package depends on.
        .target(
            name: "MyAwesomeProject",
            dependencies: ["PerfectHTTPServer"]),
    ]
)
```

As you can see, the version for this dependency is set to 3.0.0 or higher. This is the latest version of the package as of this writing and is the version which explicitly supports Swift 4. You can also see that "PerfectHTTPServer" was added to the direct dependencies list for your "MyAwesomeProject" target.

Now the project is ready to be built and run by running by the following two commands:

```
swift build
.build/debug/MyAwesomeProject
```

You will see output as SPM resolves packages and then builds the project. Running the program will produce the following:

```
Hello, world!
```

### Setting up the server

Now that the Swift package has been created and compiles, the next step is to implement the Perfect-HTTPServer. Open up the `Sources/MyAwesomeProject/main.swift` file and replace its content with the following:

``` swift
import PerfectHTTP
import PerfectHTTPServer

// Register your own routes and handlers
var routes = Routes()
routes.add(method: .get, uri: "/") {
	request, response in
	response.setHeader(.contentType, value: "text/html")
	response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
		.completed()
}

do {
	// Launch the HTTP server.
	try HTTPServer.launch(
		.server(name: "www.example.ca", port: 8181, routes: routes))
} catch {
	fatalError("\(error)") // fatal error launching one of the servers
}
```

Build and run the project again with:

```
swift build
.build/debug/MyAwesomeProject
```

You will see a message as the server starts:

```
[INFO] Starting HTTP server www.example.ca on :::8181
```

The server is now running and waiting for connections. Access [http://localhost:8181/](http://127.0.0.1:8181/) to see the greeting. Hit "control-c" to terminate the server.

### Xcode

Swift Package Manager (SPM) can generate an Xcode project which can run the MyAwesomeProject server and provide full source code editing and debugging for your project. Enter the following in your terminal:

```
swift package generate-xcodeproj
```

After opening the project ensure that you have selected the executable target and selected it to run on "My Mac". Also ensure that the correct Swift toolchain is selected. You can now run and debug the server directly in Xcode. If your application will access files within your project directory, for example HTML files in a webroot folder, choose the menu item "Product > Scheme > Edit Scheme…" and in the Options tab set the "Use Custom Working Directory" to your projects folder. This will enable you to run from within Xcode and still easily access files given relative paths.



