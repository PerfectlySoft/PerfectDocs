# Static File Content

As seen in the [Routing](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/routing.md) chapter, Perfect is capable of complex dynamic routing. It is also capable of serving static content including HTML, images, CSS, and JavaScript.

Static file content serving is handled through the ```StaticFileHandler``` object. Once a request is given to an instance of this object, it will handle finding the indicated file or returning a 404 if it does not exist. ```StaticFileHandler``` also handles caching through use of the ETag header as well as byte range serving for very large files.

A ```StaticFileHandler``` object is initialized with a document root path parameter. This root path forms the prefix to which all file paths will be appended. The current HTTPRequest's ```path``` property will be used to indicate the path to the file which should be read and returned.

```StaticFileHandler``` can be directly used by creating one in your web handler and calling its ```handleRequest``` method.

For example, a handler which simply returns the request file might look as follows:

```swift
{
	request, response in
	StaticFileHandler(documentRoot: request.documentRoot)
		.handleRequest(request: request, response: response)
}
```

If your server is intended to only server static files it can be launched directly with a document root.

```swift
try HTTPServer.launch(.server(name: "localhost", port: 8080, documentRoot: "/path/to/webroot"))
```

The following example establishes a virtual documents path, serving all URIs which begin with "/files" from the physical directory "/var/www/htdocs":

``` swift
routes.add(method: .get, uri: "/files/**") {
	request, response in

	// get the portion of the request path which was matched by the wildcard
	request.path = request.urlVariables[routeTrailingWildcardKey]

	// Initialize the StaticFileHandler with a documentRoot
	let handler = StaticFileHandler(documentRoot: "/var/www/htdocs")

	// trigger the handling of the request, 
	// with our documentRoot and modified path set
	handler.handleRequest(request: request, response: response)
}
```

In the route example above, a request to "/files/foo.html" would return the corresponding file "/var/www/htdocs/foo.html".

### Content Compression and Performance

By default, StaticFileHandler will send all file data to the client as quickly as possible. This means it will attempt to use the `sendfile` function which offers better performance when sending file data out over socket connections. There are two cases where this may not be possible or desirable. 

The first case occurs when using an encrypted TLS/SSL connection. In this scenario StaticFileHandler will simply read and send out the file data in chunks. No action is required. StaticFileHandler will detect if the data is being sent over a secured connection and will disable the use of `sendfile`.

The second case occurs when using content compression. Content compression and usage of `sendfile` do not mix and can give erratic results. To enable content compression with StaticFileHandler, `allowResponseFilters` must be set to true. This is accomplished with an additional parameter passed to the initializer:

```swift
StaticFileHandler(documentRoot: "/path/to/root/", allowResponseFilters: true)
```

