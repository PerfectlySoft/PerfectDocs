# Static File Content

As seen in the [Routing](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/routing.md) chapter, Perfect is capable of complex dynamic routing. It is also capable of serving static content including HTML, images, CSS, and JavaScript.

Static file content serving is handled through the ```StaticFileHandler``` object. Once a request is given to an instance of this object, it will handle finding the indicated file or returning a 404 if it does not exist. ```StaticFileHandler``` also handles caching through use of the ETag header as well as byte range serving for very large files.

A ```StaticFileHandler``` object is initialized with a document root path parameter. This root path forms the prefix to which all file paths will be appended. The current HTTPRequest's ```path``` property will be used to indicate the path to the file which should be read and returned.

```StaticFileHandler``` can be directly used by creating one in your web handler and calling its ```handleRequest``` method.

For example, a handler which simply returns the request file might look as follows:

```swift
{
	request, response in
	StaticFileHandler(documentRoot: request.documentRoot).handleRequest(request: request, response: response)
}
```

However, unless custom behaviour is required, it is not necessary to handle this manually. Setting the server's ```documentRoot``` property will automatically install a handler which will serve any files from the indicated directory. Setting the server's document root is analogous to the following snippet:

```swift
let dir = Dir(documentRoot)
if !dir.exists {
	try Dir(documentRoot).create()
}
routes.add(method: .get, uri: "/**", handler: {
	request, response in
	StaticFileHandler(documentRoot: request.documentRoot).handleRequest(request: request, response: response)
})
```

The handler that gets installed will serve any files from the root or any sub-directories contained therein.

An example of the documentRoot property usage is found in the PerfectTemplate:

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
		response.appendBody(string: "<html>...</html>")
		response.completed()
	}
)

// Add the routes to the server.
server.addRoutes(routes)

// Set a listen port of 8181
server.serverPort = 8181

// Set a document root.
// This is optional. If you do not want to serve 
// static content then do not set this.
// Setting the document root will automatically add a 
// static file handler for the route
server.documentRoot = "./webroot"

// Gather command line options and further configure the server.
// Run the server with --help to see the list of supported arguments.
// Command line arguments will supplant any of the values set above.
configureServer(server)

do {
	// Launch the HTTP server.
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("Network error thrown: \(err) \(msg)")
}

``` 

Note the `server.documentRoot = "./webroot"` line. It means that if there is a styles.css document in the specified webroot directory, then a request to the URI "/styles.css" will return that file to the browser.

The following example establishes a virtual documents path, serving all URIs which begin with "/files" from the physical directory "/var/www/htdocs":

``` swift
routes.add(method: .get, uri: "/files/**", handler: {
	request, response in

	// get the portion of the request path which was matched by the wildcard
	request.path = request.urlVariables[routeTrailingWildcardKey]!

	// Initialize the StaticFileHandler with a documentRoot
	let handler = StaticFileHandler(documentRoot: "/var/www/htdocs")

	// trigger the handling of the request, 
	// with our documentRoot and modified path set
	handler.handleRequest(request: request, response: response)
	}
)
```

In the route example above, a request to "/files/foo.html" would return the corresponding file "/var/www/htdocs/foo.html".

