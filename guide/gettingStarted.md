## Quick Start

### Swift 3.0

Ensure you have properly installed a Swift 3.0 toolchain from [Swift.org](https://swift.org/getting-started/). In the terminal, typing:

```
swift --version
```

should produce something like the following:

```
Apple Swift version 3.0 (swiftlang-800.0.33.1 clang-800.0.31)
Target: x86_64-apple-macosx10.9
```

### OS X
Perfect relies on [Home Brew](http://brew.sh) for installing dependencies on OS X. This is currently limited to OpenSSL. To install Home Brew, in the Terminal, type:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

To install OpenSSL:

```
brew install openssl
brew link openssl --force
```

### Linux
Perfect relies on OpenSSL, libssl-dev and uuid:

```
sudo apt-get install openssl libssl-dev uuid-dev
```

### Build Starter Project

The following will clone and build an empty starter project and launch the server on port 8181.

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
swift build
.build/debug/PerfectTemplate
```

You should see the following output:

```
Starting HTTP server on 0.0.0.0:8181 with document root ./webroot
```

This means the server is running and waiting for connections. Access [http://localhost:8181/](http://127.0.0.1:8181/) to see the greeting. Hit control-c to terminate the server.

You can view the full source code for [PerfectTemplate](https://github.com/PerfectlySoft/PerfectTemplate). 

### Xcode

Swift Package Manager can generate an Xcode project which can run the PerfectTemplate server and provide full source code editing and debugging for your project. Enter the following in your terminal:

```
swift package generate-xcodeproj
```

Open the generated file "PerfectTemplate.xcodeproj". Ensure that you have selected the executable target and selected it to run on "My Mac". You can now run and debug the server.

## Next Steps

These example snippets show how to accomplish several common tasks that one might need to do when developing a Web/REST application. In all cases, the ```request``` and ```response``` variables refer, respectively, to the ```HTTPRequest``` and ```HTTPResponse``` objects which are given to your URL handlers.

Consult the [API reference](http://www.perfect.org/docs/) for more details.

### Get a client request header

```swift
if let acceptEncoding = request.header(.acceptEncoding) {
	...
}
```

### Get client GET or POST parameters

```swift
if let foo = request.param(name: "foo") {
	...
}   
if let foo = request.param(name: "foo", defaultValue: "default foo") {
	...
}
let foos: [String] = request.params(named: "foo")
```

### Get the current request path

```swift
let path = request.path
```

### Access the server's document directory and return an image file to the client

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

### Get client cookies

```swift
for (cookieName, cookieValue) in request.cookies {
	...
}
```

### Set client cookie

```swift
let cookie = HTTPCookie(name: "cookie-name", value: "the value", domain: nil,
                    expires: .session, path: "/",
                    secure: false, httpOnly: false)
response.addCookie(cookie)
```

### Return JSON data to client

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
*This snippet uses the built-in JSON encoding. Feel free to bring in your own favorite JSON encoder/decoder.*

### Redirect the client

```swift
response.status = .movedPermanently
response.setHeader(.location, value: "http://www.perfect.org/")
response.completed()
```

### Filter and handle 404 errors in a custom manner

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