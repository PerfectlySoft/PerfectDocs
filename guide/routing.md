# Routing

Routing determines which handler receives a specific request. A handler is a routine, function, or method dedicated to receiving and acting on certain types of requests or signals. Requests are routed based on two pieces of information: the HTTP request method, and the request path. A route refers to an HTTP method, path, and handler combination. Routes are created and added to the server before it starts listening for requests. For example:

```swift
var routes = Routes()
routes.add(method: .get, uri: "/path/one") { 
	request, response in
	response.setBody(string: "Handler was called")
		.completed()
}
server.addRoutes(routes)
```

Once the Perfect server receives a request, it will pass the request object through any registered request filters. These filters have a chance to modify the request object in ways which may affect the routing process, such as changing the request path. The server will then search for a route which matches the current request method and path. If a route is successfully found for the request, the server will deliver both the request and response objects to the found handler. If a route is not found for a request, the server will send a “404” or “Not Found” error response to the client.

### Creating Routes

The routing API is part of the [PerfectHTTP](https://github.com/PerfectlySoft/Perfect-HTTP) project. Interacting with the routing system requires that you first ```import PerfectHTTP```.

Before adding any route, you will need an appropriate handler function. Handler functions accept the request and response objects, and are expected to generate content for the response. They will indicate when they have completed a task. The typealias for a request handler is as follows:

```swift
/// Function which receives request and response objects and generates content.
public typealias RequestHandler = (HTTPRequest, HTTPResponse) -> ()
```

Requests are considered active until a handler indicates that it has concluded. This is done by calling either the `HTTPResponse.next()` or the `HTTPResponse.completed()` function. Request handling in Perfect is fully asynchronous, so a handler function can return, spin off into new threads, or perform any other sort of asynchronous activity before calling either of these functions.

Request handlers can be chained, in that one request URI can identify multiple handlers along its path. The handlers will be executed in order, and each given a chance to either continue the request processing or halt it.

A request is considered to be active up until `HTTPResponse.completed()` is called. Calling `.next()` when there are no more handlers to execute is equivalent to calling `.completed()`. Once the response is marked as completed, the outgoing headers and body data, if any, will be sent to the client.

Individual uri handlers are added to a ```Routes``` object before they are added to the server. When a Routes object is created, one or more routes are added using its ```add``` functions. Routes provides the following functions:

```swift
public struct Routes {
	/// Initialize with no baseUri.
	public init(handler: RequestHandler? = nil)
	// Initialize with a baseUri.
	public init(baseUri: String, handler: RequestHandler? = nil)
	/// Add all the routes in the Routes object to this one.
	public mutating func add(routes: Routes)
	/// Add the given method, uri and handler as a route.
	public mutating func add(method: HTTPMethod, uri: String, handler: RequestHandler)
	/// Add the given method, uris and handler as a route.
	public mutating func add(method: HTTPMethod, uris: [String], handler: RequestHandler)
	/// Add the given uri and handler as a route.
	/// This will add the route got both GET and POST methods.
	public mutating func add(uri: String, handler: RequestHandler)
	/// Add the given method, uris and handler as a route.
	/// This will add the route got both GET and POST methods.
	public mutating func add(uris: [String], handler: RequestHandler)
	/// Add one Route to this object.
	public mutating func add(_ route: Route)
}
```

A Routes object can be initialized with a baseURI. The baseURI will be prepended to any route added to the object. For example, one could initialize a Routes object for version one of an API and give it a baseURI of "/v1". Every route added will be prefixed with /v1. Routes objects can also be added to other Routes objects, and each route therein will be prefixed in the same manner. The following example shows the creation of two sets of routes for two versions of an API. The second version differs in behavior in only one endpoint:

```swift
var routes = Routes()
// Create routes for version 1 API
var api = Routes()
api.add(method: .get, uri: "/call1", handler: { _, response in
	response.setBody(string: "API CALL 1")
	response.completed()
})
api.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API CALL 2")
	response.completed()
})

// API version 1
var api1Routes = Routes(baseUri: "/v1")
// API version 2
var api2Routes = Routes(baseUri: "/v2")

// Add the main API calls to version 1
api1Routes.add(routes: api)
// Add the main API calls to version 2
api2Routes.add(routes: api)
// Update the call2 API
api2Routes.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API v2 CALL 2")
	response.completed()
})

// Add both versions to the main server routes
routes.add(routes: api1Routes)
routes.add(routes: api2Routes)
```

A Routes object can also be given an optional handler. This handler will be called if that part of a path is matched against another subsequent handler.

For example, a "/v1" version of an API might enforce a particular authentication mechanism on the clients. The authentication handler could be added to the "/v1" portion of the api and as each actual endpoint is reached the authentication handler would be given a chance to evaluate the request before passing it down to the remaining handlers(s).

```swift
var routes = Routes(baseUri: "/v1") {
	request, response in 
	if authorized(request) {
		response.next()
	} else {
		response.completed(.unauthorized)
	}
}
routes.add(method: .get, uri: "/call1") { 
	_, response in
	response.setBody(string: "API CALL 1").next()
}
routes.add(method: .get, uri: "/call2") { 
	_, response in
	response.setBody(string: "API CALL 2").next()
}
```

Accessing either "/v1/call1" or "/v1/call2" will pass the request to the handler set on "/v1" routes.

Routes can be nested like this as deeply as you wish. Handlers set directly on Routes objects are considered non-terminal. That is, they can not be accessed directly by clients and will only be executed if the request goes on to match a handler which is terminal. Likewise, handlers which are terminal but lie on the path of a more complete match will not be executed. For example with a handler on "/v1/users" and on "/v1/users/foo", accessing "/v1/users/foo" will not execute "/v1/users".

### Adding Server Routes

Both Perfect-HTTPServer and Perfect-FastCGI support routing. Consult [HTTPServer]([Routing](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/HTTPServer.md) for details on how to apply routes to HTTP servers.

### Variables

Route URIs can also contain variable components. A variable component begins and ends with a set of curly brackets ```{ }```. Within the brackets is the variable identifier. A variable identifier can consist of any character except the closing curly bracket ```}```. Variable components work somewhat like single wildcards do in that they match any single literal path component value. The actual value of the URL component which is matched by the variable is saved and made available through the ```HTTPRequest.urlVariables``` dictionary. This dictionary is of type ```[String:String]```. URI variables are a good way to gather dynamic data from a request. For example, a URL might make a user management related request, and include the user id as a component in the URL.

For example, when given the URI ```/foo/{bar}/baz```, a request to the URL ```/foo/123/baz``` would match and place it in the ```HTTPRequest.urlVariables``` dictionary with the value "123" under the key "bar".

### Wildcards

A wildcard, also referred to as a wild character, is a symbol used to replace or represent one or more characters. Beyond full literal URI paths, routes can contain wildcard segments.

Wildcards match any portion of a URI and can be used to route groups of URIs to a single handler. Wildcards consist of either one or two asterisks. A single asterisk can occur anywhere in a URI path as long as it represents one full component of the URI. A double asterisk, or trailing wildcard, can occur only at the end of a URI. Trailing wildcards match any remaining portion of a URI.

A route with the the URI ```/foo/*/baz``` would match the both of these URLs:

```
/foo/123/baz
/foo/bar/baz
```

A route with the URI ```/foo/**``` would match all of the following URLs:

```
/foo/bar/baz
/foo
```

A route with the URI ```/**``` would match any request.

A trailing wildcard route will save the URI portion which is matched by the wildcard. It will place this path segment in the ```HTTPRequest.urlVariables``` map under the key indicated by the global variable ```routeTrailingWildcardKey```. For example, given the route URI "/foo/**" and a request URI "/foo/bar/baz", the following snippet would be true:

```swift
request.urlVariables[routeTrailingWildcardKey] == "/bar/baz"
```

### Priority/Ordering

Because route URIs could potentially conflict, literal, wildcard and variable paths are checked in a specific order. Path types are checked in the following order:

1. Variable paths
2. Literal paths
3. Wildcard paths
4. Trailing wildcard paths are checked last

### Implicit Trailing Wildcard

When the ```.documentRoot``` property of the server is set, the server will automatically add a ```/**``` trailing wildcard route which will enable the serving of static content from the indicated directory. For example, setting the document root to "./webroot" would permit the server to deliver any files located within that directory.

### Further Information

For more information and examples of URL routing, see the [URL Routing](https://github.com/PerfectExamples/Perfect-URLRouting) example application.
