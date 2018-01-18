# HTTPServer

This document describes the three methods by which you can launch new Perfect HTTP servers. These methods differ in their complexity and each caters to a different use case.

The first method is data driven whereby you provide either a Swift Dictionary or JSON file describing the servers you wish to launch. The second method describes the desired servers using Swift language constructs complete with the type checking and compile-time constraints provided by Swift. The third method permits you to instantiate an HTTPServer object and then procedurally configure each of the required properties before manually starting it.

HTTP servers are configured and started using the functions available in the `HTTPServer` namespace. A Perfect HTTP server consists of at least a name and a listen port, one or more handlers, and zero or more request or response filters. In addition, a secure HTTPS server will also have TLS related configuration information such as a certificate or key file path.

When starting servers you can choose to wait until the servers have terminated (which will generally not happen until the process is terminated) or receive `LaunchContext` objects for each server which permits them to be individually terminated and waited upon.

## HTTPServer Configuration Data

One or more Perfect HTTP servers can be configured and launched using structured configuration data. This includes setting elements such as the listen port and bind address but also permits pointing handlers to specific functions by name. This feature is required if loading server configuration data from a JSON file. In order to enable this functionality on Linux, you must build your SPM executable with an additional flag:

```
swift build -Xlinker --export-dynamic
```

This is only required on Linux and only if you are going to be using the configuration data system described in this section with JSON text files.

Call one of the static `HTTPServer.launch` functions with either a path to a JSON configuration file, a File object pointing to the configuration file or a Swift Dictionary&lt;String:Any&gt;.

The resulting configuration data will be used to launch one or more HTTP servers.

``` swift
public extension HTTPServer {
	public static func launch(wait: Bool = true, configurationPath path: String) throws -> [LaunchContext]
	public static func launch(wait: Bool = true, configurationFile file: File) throws -> [LaunchContext]
	public static func launch(wait: Bool = true, configurationData data: [String:Any]) throws -> [LaunchContext]
}
```

The default value for the `wait` parameter indicates that the function should not return but should block until all servers have terminated or the process is killed. If `false` is given for `wait` then the returned array of `LaunchContext` objects can be used to monitor or terminate the individual servers. Most applications will want the function to wait and so the functions can be called without including the `wait` parameter.

``` swift
do {
	try HTTPServer.launch(configurationPath: "/path/to/perfecthttp.json")
} catch {
	// handle critical failure
}
```

Note that the configuration file can be located or named whatever you'd like, but it should have the `.json` file extension. We may support other file formats in the future and ensuring that your configuration file has an extension which describes its content is important.

After it is decoded from JSON, at its top level, the configuration data should contain a "servers" key with a value that is an array of Dictionary&lt;String:Any&gt;. These dictionaries describe the servers which will be launched.

``` swift
[
	"servers":[
		[…],
		[…],
		[…]
	]
]
```

A simple example single server configuration dictionary might look as follows. Note that the keys and values in this example are all explained in the subsequent sections of this document.

``` swift
[
	"servers":[
		[
			"name":"localhost",
			"port":8080,
			"routes":[
				[
					"method":"get",
					"uri":"/**",
					"handler":PerfectHTTPServer.HTTPHandler.staticFiles,
					"documentRoot":"/path/to/webroot"
				],
				[
					"methods":["get", "post"],
					"uri":"/api/**",
					"handler":PerfectHTTPServer.HTTPHandler.redirect,
					"base":"http://other.server.ca"
				]
			]
		]
	]
]
```

The options available for servers are as follows:

### name:

This **required** string value is primarily used to identify the server and would generally be the same as the server's domain name. Applications may use the server name to construct URLs pointing back to the server.

It is permitted to use the same name for multiple servers. For example, you may have three servers on the same host all listening on different ports. These three servers could all have the same name.

Corresponding HTTPServer property: `HTTPServer.serverName`.

### port:

This **required** integer value indicates the port on which the server should listen. TCP ports range from 0 to 65535. Ports in the range of 0 to 1024 require root permissions to start. See the *runAs* option to indicate the user which the process should switch to after binding on such a port.

Corresponding HTTPServer property: `HTTPServer.serverPort`.

### address:

This **optional** String value should be a dotted IP address. This indicates the local address on which the server should bind. If not given, this value defaults to "0.0.0.0" which indicates that the server should bind on all available local IP addresses. Using "::" for this value will enable listening on all local IPv6 &amp; IPv4 addresses.

Corresponding HTTPServer property: `HTTPServer.serverAddress`.

### routes:

This **optional** element should have a value that is an array of [String:Any] dictionaries. Each element of the array indicates a URI route or group of URI toues which map an incoming HTTP request to a handler. See [Routing](routing.md) for specifics on Perfect's URI routing system.

Each [String:Any] dictionary in the "routes" array must be either a simple route with a handler or a group of routes.

If the dictionary has a "uri" and a "handler" key, then it is assumed to be a simple route. A group of yours must have at least a "children" key.

A simple route consists of zero or more HTTP methods, a URI and the name of a `RequestHandler` function or a function which returns a `RequestHandler`. The key names are: "method" or "methods", "uri", and "handler". The value for the "methods" key should be an array of strings. If no method values are provided then any HTTP method may trigger the handler.

A group of handlers consists of an optional "baseURI", an optional "handler", and a required array of "children" whose value must be [[String:Any]]. Each of the children in this array can in turn express either simple route handlers or further groups of routes. The optional "handler" function will be executed before the final request handler. Multiple handlers can be chained in this manner, some running earlier to set up certain state or to screen the incomming requests. Handlers used in this manner should call `response.next()` to indicate that the handler has finished executing and the next one can be called. If there are no further handlers then the request will be completed. If an intermediate handler determines that the request should go no futher, it can call `response.completed()` and no futher handlers will be executed.

Perfect comes with request handlers that take care of various common tasks such as redirecting clients or serving static, on-disk files. The following example defines a server which listens on port 8080 and has two handlers, one of which serves static files while the other redirects clients to a new URL.

``` swift
[
	"servers":[
		[
			"name":"localhost",
			"port":8080,
			"routes":[
				[
					"method":"get",
					"uri":"/**",
					"handler":PerfectHTTPServer.HTTPHandler.staticFiles,
					"documentRoot":"/path/to/webroot"
				],
				[
					"baseURI":"/api",
					"children":[
						[
							"methods":["get", "post"],
							"uri":"/**",
							"handler":PerfectHTTPServer.HTTPHandler.redirect,
							"base":"http://other.server.ca"
						]
					]
				]
			]
		]
	]
]
```

Corresponding HTTPServer property: `HTTPServer.addRoutes`.

#### Adding Custom Request Handlers

While the built-in Perfect request handlers can be handy, most developers will want to add custom behaviour to their servers. The "handler" key values can point to your own functions which will each return the `RequestHandler` to be called when the route uri matches an incoming request.

It's important to note that the function names which you would enter into the configuration data are **static** functions which *return* the `RequestHandler` that will be subsequently used. These functions accept the current configuration data [String:Any] for the particular route in order to extract any available configuration data such as the `staticFiles` "documentRoot" described above.

Alternatively, if you do not need any of the available configuration data (for example, your handler requires no configuration) you can simply indicate the `RequestHandler` itself.

It's also vital that the name you provide be fully qualified. That is, it should include your Swift module name, the name of any interstitial nesting constructs such as struct or enum, and then the function name itself. These should all be separated by "." (periods). For example, you can see the static file handler is given as "PerfectHTTPServer.HTTPHandler.staticFiles". It resides in the module "PerfectHTTPServer", in an extension of the struct "HTTPHandler" and is named "staticFiles".

Note that if you are creating a configuration directly in Swift code as a dictionary then you do not have to quote the function names that you provide. The value for the "handler" (and subsequently the "filters" described later in this chapter) can be given as direct function references.

An example request handler generator which could be used in a server configuration follows.

``` swift
public extension HTTPHandler {
	public static func staticFiles(data: [String:Any]) throws -> RequestHandler {
		let documentRoot = data["documentRoot"] as? String ?? "./webroot"
		let allowResponseFilters = data["allowResponseFilters"] as? Bool ?? false
		return {
			req, resp in
			StaticFileHandler(documentRoot: documentRoot, allowResponseFilters: allowResponseFilters)
				.handleRequest(request: req, response: resp)
		}
	}
}
```

Note: the `HTTPHandler` struct is an abstract namespace defined in PerfectHTTPServer. It consists of only static request handler generators such as this.

Request handler generators are encouraged to `throw` when required configuration data is not provided by the user or if the data is invalid. Ensure that the Error you throw will provide a helpful message when it is converted to String. This will ensure that users see such configuration problems early so that they can be corrected. If the handler generator cannot return a valid `RequestHandler` then it should throw an error.

### filters:

Request filters can screen or manipulate incoming request data. For example, an authentication filter might check to see if a request has certain permissions, and if not, return an error to the client. Response filters do the same for outgoing data, having an opportunity to change response headers or body data. See [Request and Response Filters](filters.md) for specifics on Perfect's request filtering system.

The value for the "filters" key is an array of dictionaries containing keys which describe each filter. The required keys for these dictionaries are "type", and "name". The possible values for the "type" key are "request" or "response", to indicate either a request or a response filter. A "priority" key can also be provided with a value of either "high", "medium", or "low". If a priority is not provided then the default value will be "high".

The following example adds two filters, one for requests and one for responses.

``` swift
[
	"servers": [
		[
		"name":"localhost",
		"port":8080,
		"routes":[
			["method":"get", "uri":"/**", "handler":"PerfectHTTPServer.HTTPHandler.staticFiles",
			"documentRoot":"./webroot"]
		],
		"filters":[
			[
				"type":"request",
				"priority":"high",
				"name":"PerfectHTTPServer.HTTPFilter.customReqFilter"
			],
			[
				"type":"response",
				"priority":"high",
				"name":"PerfectHTTPServer.HTTPFilter.custom404",
				"path":"./webroot/404.html"
			]
		]
	]
]
```

Filter names work in much the same way as route handlers do, however, the function signatures are different. A request filter generator function takes the [String:Any] containing the configuration data and returns a `HTTPRequestFilter` or a `HTTPResponseFilter` depending on the filter type.

``` swift
// a request filter generator
public func customReqFilter(data: [String:Any]) throws -> HTTPRequestFilter {
	struct ReqFilter: HTTPRequestFilter {
		func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
			callback(.continue(request, response))
		}
	}
	return ReqFilter()
}

// a response filter generator
public func custom404(data: [String:Any]) throws -> HTTPResponseFilter {
	guard let path = data["path"] as? String else {
		fatalError("HTTPFilter.custom404(data: [String:Any]) requires a value for key \"path\".")
	}
	struct Filter404: HTTPResponseFilter {
		let path: String
		func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
			if case .notFound = response.status {
				do {
					response.setBody(string: try File(path).readString())
				} catch {
					response.setBody(string: "An error occurred but I could not find the error file. \(response.status)")
				}
				response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
			}
			return callback(.continue)
		}
		func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
			callback(.continue)
		}
	}
	return Filter404(path: path)
}
```

Corresponding HTTPServer properties: `HTTPServer.setRequestFilters`, `HTTPServer.setResponseFilters`.

### Sharing Information Among Request, Filter and Response

Sometimes it is necessary to share certain information among HTTPRequest, HTTPFiler and HTTPResponse. For example, an authentication filter may decode a user id from cookies or JWT tokens and make the use profile available to the request session scope as a typical security design. 

Although HTTP is stateless and blocking the transparency between filters and requests, Perfect HTTPServer still provides an indirect way to share these valuable information in each request and make the whole design more efficient:

``` swift
/// an authentication filter, which intercept all incoming request
/// and assuming all successful login should present a token.
func filter(request: HTTPRequest, response: HTTPResponse,
                     callback: (HTTPRequestFilterResult) -> ()) {
  guard let token = request.param("token"), 
  let userId = token.decode(),
  let profile = database.load(userId) as? Profile else {
  	// the request doesn't hold a valid token
  	// access denied
  	response.status = .forbidden
  	response.completed
  	callback(.halt(request, response))
  }
  
  	// user information retrieved successfully, 
  	// now save this info into the memory
  response.request.scratchPad["currentUserProfile"] = profile
  	
  // then invoke the following request
  callback(.continue(request, response))
}

routes.add(Route(method: .get, uri: "/api/aboutUser", handler: {
      request, response in
      guard let profile 
      	= response.request.scratchPad["currentUserProfile"] as? Profile
		else {
      		//something wrong
      	}
      	/// now you can directly use the profile
    }))
```

In the above example, the priority filter intercepts the request first and translates the request into a user profile, then passes the user profile to the following requests, which implements a centralized authentication middleware.

### tlsConfig:

If a "tlsConfig" key is provided then a secure HTTPS server will be attempted. The value for the TLS config should be a dictionary containing the following required and optional keys/values:

* certPath - **required** String file path to the certificate file
* keyPath - optional String file path to the key file
* cipherList - optional array of ciphers that the server will support
* caCertPath - optional String file path to the CA cert file
* verifyMode - optional String indicating how the secure connections should be verified. The value should be one of:
	* none
	* peer
	* failIfNoPeerCert
	* clientOnce
	* peerWithFailIfNoPeerCert
	* peerClientOnce
	* peerWithFailIfNoPeerCertClientOnce

The default values for the cipher list can be obtained through the `TLSConfiguration.defaultCipherList` property.

### User Switching

After starting as root and binding the servers to the indicated ports (low, restricted ports such as 80, for example), it is recommended that the server process switch to a non-root operating system user. These users are generally given low or restricted permissions in order to prevent security attacks which could be perpetrated were the server running as root.

At the top level of your configuration data (as a sibling to the "servers" key), you can include a "runAs" key with a string value. This value indicates the name of the desired user. The process will switch to the user only after all servers have successfully bound their respective listen ports.

Only a server process which is started as root can switch users.

Corresponding HTTPServer function: `HTTPServer.runAs(_ user: String)`.

## HTTPServer.launch

There are several variants of the `HTTPServer.launch` functions which permit one or more servers to be started. These functions abstract the inner workings of the HTTPServer object and provide a more streamlined interface for server launching.

The simplest of these methods launches a single server with options:

``` swift
public extension HTTPServer {
	public static func launch(wait: Bool = true, name: String, port: Int, routes: Routes,
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
	public static func launch(wait: Bool = true, name: String, port: Int, routes: [Route],
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
	public static func launch(wait: Bool = true, name: String, port: Int, documentRoot root: String,
	                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
	                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) throws -> LaunchContext
}
```

The remaining launch functions take one or more server descriptions, launches them and returns their `LaunchContext` objects.

``` swift
public extension HTTPServer {
	public static func launch(wait: Bool = true, _ servers: [Server]) throws -> [LaunchContext]
	public static func launch(wait: Bool = true, _ server: Server, _ servers: Server...) throws -> [LaunchContext]
}
```

The `Server`, which describes the HTTPServer object that will eventually be launched, looks like so:

``` swift
public extension HTTPServer {
	public struct Server {
		public init(name: String, address: String, port: Int, routes: Routes,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(tlsConfig: TLSConfiguration, name: String, address: String, port: Int, routes: Routes,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(name: String, port: Int, routes: Routes,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])
		public init(tlsConfig: TLSConfiguration, name: String, port: Int, routes: Routes,
		            requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		            responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [])

		public static func server(name: String, port: Int, routes: Routes,
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func server(name: String, port: Int, routes: [Route],
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func server(name: String, port: Int, documentRoot root: String,
		                          requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                          responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
		public static func secureServer(_ tlsConfig: TLSConfiguration, name: String, port: Int, routes: [Route],
		                                requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [],
		                                responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = []) -> Server
	}
}
```

The following examples show some common usages.

``` swift
// start a single server serving static files
try HTTPServer.launch(name: "localhost", port: 8080, documentRoot: "/path/to/webroot")

// start two servers. have one serve static files and the other handle API requests
let apiRoutes = Route(method: .get, uri: "/foo/bar", handler: {
		req, resp in
		//do stuff
	})
try HTTPServer.launch(
	.server(name: "localhost", port: 8080, documentRoot:  "/path/to/webroot"),
	.server(name: "localhost", port: 8181, routes: [apiRoutes]))

// start a single server which handles API and static files
try HTTPServer.launch(name: "localhost", port: 8080, routes: [
	Route(method: .get, uri: "/foo/bar", handler: {
		req, resp in
		//do stuff
	}),
	Route(method: .get, uri: "/foo/bar", handler:
		HTTPHandler.staticFiles(documentRoot: "/path/to/webroot"))
	])

let apiRoutes = Route(method: .get, uri: "/foo/bar", handler: {
		req, resp in
		//do stuff
	})
// start a secure server
try HTTPServer.launch(.secureServer(TLSConfiguration(certPath: "/path/to/cert"), name: "localhost", port: 8080, routes: [apiRoutes]))
```

The `TLSConfiguration` struct configures the server for HTTPS and is defined as:

```swift
public struct TLSConfiguration {
	public init(certPath: String, keyPath: String? = nil,
	            caCertPath: String? = nil, certVerifyMode: OpenSSLVerifyMode? = nil,
	            cipherList: [String] = TLSConfiguration.defaultCipherList)
}
```

## LaunchContext

If `wait: false` is given to any of the `HTTPServer.launch` functions then one or more `LaunchContext` objects are returned. These objects permit each server's status to be checked and permit the server to be terminated.

``` swift
public extension HTTPServer {
	public struct LaunchFailure: Error {
		let message: String
		let configuration: Server
	}

	public class LaunchContext {
		public var terminated: Bool
		public let server: Server
		public func terminate() -> LaunchContext
		public func wait(seconds: Double = Threading.noTimeout) throws -> Bool
	}
}
```

If a launched server fails because an error is thrown then that error will be translated and thrown when the `wait` function is called.

## HTTPServer Object

An `HTTPServer` object can be instantiated, configured and manually started.

``` swift
public class HTTPServer {
	/// The directory in which web documents are sought.
	/// Setting the document root will add a default URL route which permits
	/// static files to be served from within.
	public var documentRoot: String
	/// The port on which the server is listening.
	public var serverPort: UInt16 = 0
	/// The local address on which the server is listening. The default of 0.0.0.0 indicates any address.
	public var serverAddress = "0.0.0.0"
	/// Switch to user after binding port
	public var runAsUser: String?

	/// The canonical server name.
	/// This is important if utilizing the `HTTPRequest.serverName` property.
	public var serverName = ""
	public var ssl: (sslCert: String, sslKey: String)?
	public var caCert: String?
	public var certVerifyMode: OpenSSLVerifyMode?
	public var cipherList: [String]
	/// Initialize the server object.
	public init()
	/// Add the Routes to this server.
	public func addRoutes(_ routes: Routes)
	/// Set the request filters. Each is provided along with its priority.
	/// The filters can be provided in any order. High priority filters will be sorted above lower priorities.
	/// Filters of equal priority will maintain the order given here.
	public func setRequestFilters(_ request: [(HTTPRequestFilter, HTTPFilterPriority)]) -> HTTPServer
	/// Set the response filters. Each is provided along with its priority.
	/// The filters can be provided in any order. High priority filters will be sorted above lower priorities.
	/// Filters of equal priority will maintain the order given here.
	public func setResponseFilters(_ response: [(HTTPResponseFilter, HTTPFilterPriority)]) -> HTTPServer
	/// Start the server. Does not return until the server terminates.
	public func start() throws
	/// Stop the server by closing the accepting TCP socket. Calling this will cause the server to break out of the otherwise blocking `start` function.
	public func stop()
}
```
