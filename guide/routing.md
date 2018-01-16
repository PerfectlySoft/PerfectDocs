# Routing

Routing determines which handler receives a specific request. A handler is a function dedicated to receiving and acting on certain requests. Requests are routed based on two pieces of information: the HTTP request method, and the request path. A route refers to an HTTP method, path, and handler combination. Routes are created and added to the server before it starts listening for requests. For example:

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

A Routes object can be initialized with a baseURI. The baseURI will be prepended to any route added to the object. For example, one could initialize a Routes object for version one of an API and give it a baseURI of "/v1". Every route added will be prefixed with /v1. Routes objects can also be added to other Routes objects, and each route therein will be prefixed in the same manner. The following example shows the creation of two sets of routes for two versions of an API. The second version differs in behaviour in only one endpoint:

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

### <a name="typed"></a>Typed Routing

In addition to raw handlers which accept request and response objects, Perfect-HTTP routes also support strongly typed handlers which decode, accept, or return Codable Swift objects.

The APIs for working with typed routes are very similar to those for working with un-typed routes. The objects for producing typed routes are named `TRoutes` and `TRoute`. The meaning and usage of these objects correspond closely to those of the `Routes` and `Route` objects, respectively. 

The interfaces for these objects are as follows:

```swift
/// A typed intermediate route handler parameterized on the input and output types.
public struct TRoutes<I, O> {
	/// Input type alias
	public typealias InputType = I
	/// Output type alias
	public typealias OutputType = O
	/// Init with a base URI and handler.
	public init(baseUri u: String,
				handler t: @escaping (InputType) throws -> OutputType)
	/// Add a typed route to this base URI.
	@discardableResult
	public mutating func add<N>(_ route: TRoute<OutputType, N>) -> TRoutes
	/// Add other intermediate routes to this base URI.
	@discardableResult
	public mutating func add<N>(_ route: TRoutes<OutputType, N>) -> TRoutes
	/// Add a route to this object. The new route will take the output of this route as its input.
	@discardableResult
	public mutating func add<N: Codable>(method m: HTTPMethod,
										 uri u: String,
										 handler t: @escaping (OutputType) throws -> N) -> TRoutes
}

/// A typed route handler.
public struct TRoute<I, O: Codable> {
	/// Input type alias.
	public typealias InputType = I
	/// Output type alias.
	public typealias OutputType = O
	// Init with a method, uri, and handler.
	public init(method m: HTTPMethod,
				uri u: String,
				handler t: @escaping (InputType) throws -> OutputType)
	/// Init with zero or more methods, a uri, and handler.
	public init(methods m: [HTTPMethod] = [.get, .post],
				uri u: String,
				handler t: @escaping (InputType) throws -> OutputType)
}
```

Just as with `Routes` and `Route` objects, a `TRoutes` is an intermediate handler and a `TRoute` is a terminal handler.

A `TRoutes` handler can be created accepting either an HTTPRequest object or any other type of object which may be passed down from a previous handler. The first `TRoutes` handler is usually created accepting the HTTPRequest object. This handler in turn processes its input and returns some object which is given to subsequent handlers.

A `TRoute` handler accepts some sort of input type and returns a Codable object. This codable object is serialized to JSON and returned to the client.

The input to both types of handlers will either be the HTTPRequest, the result of decoding the request body to some Decodable object, or the return value of whatever intermediate handler occurred immediately before. When a handler is defined as receiving a Decodable object, the HTTPRequest body will be automatically decoded into this type. If the body can not be decoded then an Error will be thrown and the error response will be returned to the client. Alternatively, a handler can be defined as accepting the HTTPRequest object but can decode the body itself using the `HTTPRequest.decode` function (described below).

#### Request Body Decode

Two extensions on the HTTPRequest object aid in decoding the request body. 

```swift
/// Extensions on HTTPRequest which permit the request body to be decoded to a Codable type.
public extension HTTPRequest {
	/// Decode the request as the desired object.
	func decode<A: Codable>() throws -> A
	/// Identity decode. Used to permit generic code to operate with the HTTPRequest
	func decode() throws -> Self
}
```

The first function will decode the body into the desired Codable object. If the request's content-type is application/json then the body will be decoded from that JSON. Otherwise, the request's URL encoded GET or POST arguments will be used for the decode. Additionally, any URL variables (described later in this document) will be utilized for the decode. This allows for a mixture of GET/POST arguments and URL variables to be brought together when decoding the object.

Note that when decoding objects from non-JSON request data, nested, non-integral objects are not supported. Objects with array properties are also not supported in this case. 

#### Response Error

If either an intermediate or terminal typed handler experiences an error during processing, they can throw an `HTTPResponseError`. Initializing one of these objects requires both an `HTTPResponseStatus` and a String description of the problem. 

```swift
/// A codable response type indicating an error.
public struct HTTPResponseError: Error, Codable, CustomStringConvertible {
	/// The HTTP status for the response.
	public let status: HTTPResponseStatus
	/// Textual description of the error.
	public let description: String
	/// Init with status and description.
	public init(status s: HTTPResponseStatus,
				description d: String)
}
```

#### Support Extensions

Extensions on `Routes` permits adding `TRoutes` or `TRoute` objects.

```swift
public extension Routes {
	/// Add routes to this object.
	mutating func add<I, O>(_ route: TRoutes<I, O>)
	/// Add a route to this object.
	mutating func add<I, O>(_ route: TRoute<I, O>)
}
```

#### <a name="typed_examples"></a>Typed Routing Examples

The following example shows how Codable objects for a route would be defined and how the typed routes would be added to a `Routes` object.

In this abbreviated example the intermediate handler for "/api" would perform some screening on the request to ensure the client has been authenticated. It would then return to the next handler (which is terminal, in this case) a tuple consisting of the original HTTPRequest object as well as a `SessionInfo` object containing whatever client id had been pulled from the request. The terminal handler "/api/info/{id}" would then use this information to complete the request and return the response.

```swift
struct SessionInfo: Codable {
	//...could be an authentication token, etc.
	let id: String
}
struct RequestResponse: Codable {
	struct Address: Codable {
		let street: String
		let city: String
		let province: String
		let country: String
		let postalCode: String
	}
	let fullName: String
	let address: Address
}
// when handlers further down need the request you can pass it along. this is not necessary though
typealias RequestSession = (request: HTTPRequest, session: SessionInfo)

// intermediate handler for /api
func checkSession(request: HTTPRequest) throws -> RequestSession {
	// one would check the request to make sure it's authorized
	let sessionInfo: SessionInfo = try request.decode() // will throw if request does not include id
	return (request, sessionInfo)
}

// terminal handler for /api/info/{id}
func userInfo(session: RequestSession) throws -> RequestResponse {
	// return the response for this request
	return .init(fullName: "Justin Trudeau",
				 address: .init(street: "111 Wellington St",
								city: "Ottawa",
								province: "Ontario",
								country: "Canada",
								postalCode: "K1A 0A6"))
}
// root Routes object holding all other routes for this server
var routes = Routes()
// types routes object for the /api URI
var apiRoutes = TRoutes(baseUri: "/api", handler: checkSession)
// add terminal handler for the /info/{id} URI suffix
apiRoutes.add(method: .get, uri: "/info/{id}", handler: userInfo)
// add the typed routes to the root
routes.add(apiRoutes)

//... go on to add routes to server
```

### Further Information

For more information and examples of URL routing, see the [URL Routing](https://github.com/PerfectExamples/Perfect-URLRouting) example application.
