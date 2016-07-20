# Static File Content

As seen in the Routing chapter, Perfect is capable of complex dynamic routing. It is also capable of serving static content such as html, images, css, and javascript just to name a few.

Static file serving can be handled in two ways: through a blanket `documentRoot` route; and specifically added routes that can include other processes like authentication verification.

An example of the "blanket" documentRoot statement usage is found in the PerfectTemplate:

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

Note the `server.documentRoot = "./webroot"` line: this means that if there is a styles.css document in the webroot directory specified, then a request to the webserver to "/styles.css" will return that file to the browser.

Alternatively, adding a route to serve a specific file or group of files from a specific directory:

``` swift
routes.add(method: .get, uri: "/static", handler: {
	request, response in

	// set the path to something specific, 
	// or trim the "/static" from the path
	request.path = "404.html"
	
	// Initialize the StaticFileHandler with a documentRoot
	let handler = StaticFileHandler(documentRoot: "/path/to/httpdocs")

	// trigger the handling of the request, 
	// with our documentRoot and modified path set
	handler.handleRequest(request: request, response: response)
	
	// signal to the HTTPResponse object that the route is finished
	response.completed()
	}
)

```

In the route example above, a request to "/static" will return the 404.html file contained in the specified directory.