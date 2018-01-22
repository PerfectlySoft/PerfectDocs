# Getting Started

Are you eager to get programming with Swift and Perfect? This guide will provide you with everything you need to know to run Perfect, and to create your first application.

After reading this guide, you will know:

- How to create and run an HTTP/HTTPS server and get Perfect up and running
- The prerequisite components you must install to run Perfect on either OS X or Ubuntu Linux
- How to build, test, and manage dependencies for Swift projects
- How to deploy Perfect in additional environments including Heroku, Amazon Web Services, Docker, Microsoft Azure, Google Cloud, IBM Bluemix CloudFoundry, and IBM Bluemix Docker

## Prerequisites

### Swift 4.0

After you have installed a Swift 4.0 toolchain from [Swift.org](https://swift.org/getting-started/), open up a terminal window and type
```
swift --version
```

It will produce a message similar to this one:

```
Apple Swift version 4.0 (swiftlang-900.0.65 clang-900.0.37)
Target: x86_64-apple-macosx10.9
```
Make sure you are running the release version of Swift 4.0+. Perfect will not compile successfully if you are running a version of Swift that is lower than 3.0.1.

You can find out which version of Swift you will need by looking in [the README of the main Perfect repo](https://github.com/PerfectlySoft/Perfect#compatibility-with-swift).

### macOS

Everything you need is already installed.

### Ubuntu Linux

Perfect runs in Ubuntu Linux 16.04 environments. Perfect relies on OpenSSL, libssl-dev, and uuid-dev. To install these, in the terminal, type:

```
sudo apt-get install openssl libssl-dev uuid-dev
```

When building on Linux, OpenSSL 1.0.2+ is required for this package. On Ubuntu 14 or some Debian distributions you will need to update your OpenSSL before this package will build.

## Getting Started with Perfect

Now you’re ready to build your first web application starter project.

### Build Starter Project

The following will clone and build an empty starter project. It will launch a local server that will run on port 8181 on your computer:

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
.build/debug/PerfectTemplate
```

You should see the following output:

```
[INFO] Starting HTTP server localhost on 0.0.0.0:8181
```

The server is now running and waiting for connections. Access [http://localhost:8181/](http://127.0.0.1:8181/) to see the greeting. Hit "control-c" to terminate the server.

You can view the full source code for [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate).

### Xcode

Swift Package Manager (SPM) can generate an Xcode project which can run the PerfectTemplate server and provide full source code editing and debugging for your project. Enter the following in your terminal:

```
swift package generate-xcodeproj
```

Open the generated file "PerfectTemplate.xcodeproj". Ensure that you have selected the executable target and selected it to run on "My Mac". You can now run and debug the server directly in Xcode.

## Next Steps

These example snippets show how to accomplish several common tasks that one might need to do when developing a web or REST application. In all cases, the ```request``` and ```response``` variables refer, respectively, to the ```HTTPRequest``` and ```HTTPResponse``` objects which are given to your URL handlers.

Consult the [API reference](https://perfect.org/docs/api.html) for more details.

### Get a Client Request Header

```swift
if let acceptEncoding = request.header(.acceptEncoding) {
	...
}
```

### Client GET and POST Parameters

```swift
if let foo = request.param(name: "foo") {
	...
}
if let foo = request.param(name: "foo", defaultValue: "default foo") {
	...
}
let foos: [String] = request.params(named: "foo")
```

### Get the Current Request Path

```swift
let path = request.path
```

### Accessing the Server's Document Directory and Returning an Image File to the Client

```swift
let docRoot = request.documentRoot
do {
    let mrPebbles = File("\(docRoot)/mr_pebbles.jpg")
    let imageSize = mrPebbles.size
    let imageBytes = try mrPebbles.readSomeBytes(count: imageSize)
    response.setHeader(.contentType, value: MimeType.forExtension("jpg"))
    response.setHeader(.contentLength, value: "\(imageBytes.count)")
    response.setBody(bytes: imageBytes)
} catch {
    response.status = .internalServerError
    response.setBody(string: "Error handling request: \(error)")
}
response.completed()
```

### Getting Client Cookies

```swift
for (cookieName, cookieValue) in request.cookies {
	...
}
```

### Setting Client Cookies

```swift
let cookie = HTTPCookie(name: "cookie-name", value: "the value", domain: nil,
                    expires: .session, path: "/",
                    secure: false, httpOnly: false)
response.addCookie(cookie)
```

### Returning JSON Data to Client

```swift
response.setHeader(.contentType, value: "application/json")
let d: [String:Any] = ["a":1, "b":0.1, "c": true, "d":[2, 4, 5, 7, 8]]

do {
    try response.setBody(json: d)
} catch {
    //...
}
response.completed()
```
*This snippet uses built-in JSON encoding. Feel free to use the JSON encoder/decoder you prefer.*

### Redirecting the Client

```swift
response.status = .movedPermanently
response.setHeader(.location, value: "http://www.perfect.org/")
response.completed()
```

### Filtering and Handling 404 Errors in a Custom Manner

```swift
struct Filter404: HTTPResponseFilter {
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}

	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		if case .notFound = response.status {
			response.bodyBytes.removeAll()
			response.setBody(string: "The file \(response.request.path) was not found.")
			response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
			callback(.done)
		} else {
			callback(.continue)
		}
	}
}

try HTTPServer(documentRoot: webRoot)
	.setResponseFilters([(Filter404(), .high)])
	.start(port: 8181)
```
