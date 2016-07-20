# Request &amp; Response Filters

In addition to the regular request/response handler system, Perfect Server also provides a request and response filtering system. Any filters which are added to the server are called for each client request. These filters run, each in turn, and are given a chance to modify either the request object before it is delivered to the handler or the response object after the request has been marked as complete. Filters also have the option to terminate the current request.

Filters are added to the server along with a priority indicator. Priority levels can be either high, medium or low. High priority filters are executed before medium and low. Medium priorities are executed before any low level filters.

Because filters are executed for every request, it is vital that they perform their tasks as quickly as possible so as to not hold up or delay request processing.

## Request Filters

Request filters are called after the request has been fully read but before the appropriate request handler has been located. This gives request filters an opportunity to modify the request before it is handled.

### Creating

Request filters must conform to the ```HTTPRequestFilter``` protocol:

```swift
/// A filter which can be called to modify a HTTPRequest.
public protocol HTTPRequestFilter {
	/// Called once after the request has been read but before any handler is executed.
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ())
}
```

When it comes time for the filter to run, its ```filter``` function will be called. The filter should perform any activities it needs and then call the provided callback to indicate that it has completed its processing. The callback takes a value which indicates what the next step should be. This can indicate that the system should either continue with processing filters, stop processing request filters at the current priority level and proceed with delivering the request to a handler, or terminate the request entirely.

```swift
/// Result from one filter.
public enum HTTPRequestFilterResult {
	/// Continue with filtering.
	case `continue`(HTTPRequest, HTTPResponse)
	/// Halt and finalize the request. Handler is not run.
	case halt(HTTPRequest, HTTPResponse)
	/// Stop filtering and execute the request.
	/// No other filters at the current priority level will be executed.
	case execute(HTTPRequest, HTTPResponse)
}
```

Because the filter receives both the request and response objects and then delivers request and response objects in its ```HTTPRequestFilterResult```, it's possible for a filter to entirely replace these objects if desired.

### Adding

Request filters are set directly on the server and given as an array of filter and priority tuples.

```swift
public class HTTPServer {
	public func setRequestFilters(_ request: [(HTTPRequestFilter, HTTPFilterPriority)]) -> HTTPServer
}
```

Calling this function sets the server's request filters. Each filter is provided along with its priority. The filters in the array parameter can be given in any order. The server will sort them appropriately, putting high priority filters above those with lower priorities. Filters of equal priority will maintain the given order.

### Example

The following example is taken from a filter related test case and illustrates how to create and add filters and shows how the filter priority levels interact.

```swift
var oneSet = false
var twoSet = false
var threeSet = false
	
struct Filter1: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		oneSet = true
		callback(.continue(request, response))
	}
}
struct Filter2: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(oneSet)
		XCTAssert(!twoSet && !threeSet)
		twoSet = true
		callback(.execute(request, response))
	}
}
struct Filter3: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(false, "This filter should be skipped")
		callback(.continue(request, response))
	}
}
struct Filter4: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(oneSet && twoSet)
		XCTAssert(!threeSet)
		threeSet = true
		callback(.halt(request, response))
	}
}

var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
		request, response in
		XCTAssert(false, "This handler should not execute")
		response.completed()
	}
)

let requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [
	(Filter1(), HTTPFilterPriority.high), 
	(Filter2(), HTTPFilterPriority.medium), 
	(Filter3(), HTTPFilterPriority.medium), 
	(Filter4(), HTTPFilterPriority.low)
]

let server = HTTPServer()
server.setRequestFilters(requestFilters)
server.serverPort = 8181
server.addRoutes(routes)
try server.start()

```

## Response Filters

Response filters are executed each once before response header data is sent to the client and each once for any subsequent chunk of body data. These filters can modify the outgoing response in any way they see fit, including adding or removing headers or rewriting body data.

### Creating

Response filters must conform to the ```HTTPResponseFilter``` protocol.

```swift
/// A filter which can be called to modify a HTTPResponse.
public protocol HTTPResponseFilter {
	/// Called once before headers are sent to the client.
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ())
	/// Called zero or more times for each bit of body data which is sent to the client.
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ())
}
```

When it comes time to send response headers the ```filterHeaders``` function is called. This function should perform whatever tasks it needs on the provided ```HTTPResponse``` object and then call the callback function. It should deliver unto the callback one of the ```HTTPResponseFilterResult``` values, which are defined as follows:

```swift
/// Response from one filter.
public enum HTTPResponseFilterResult {
	/// Continue with filtering.
	case `continue`
	/// Stop executing filters until the next push.
	case done
	/// Halt and close the request.
	case halt
}
```

These values indicate if the system should continue processing filters, stop executing filters until the next data push, or halt and terminate the request entirely.

When it comes time to send out one descrete chunk of data to the client, the filters' ```filterBody``` function is called. This function can inspect the outgoing data in the ```HTTPResponse.bodyBytes``` property and potentially modify or replace the data. Note that since the headers have already been pushed out at this stage any modifications to header data will be ignored. Once a filter's body filtering has concluded it should call the provided callback and deliver a ```HTTPResponseFilterResult```. The meaning of these values is the same as for the ```filterHeaders``` function.

### Adding

Response filters are set directly on the server and given as an array of filter and priority tuples.

```swift
public class HTTPServer {
	public func setResponseFilters(_ response: [(HTTPResponseFilter, HTTPFilterPriority)]) -> HTTPServer
}
```

Calling this function sets the server's response filters. Each filter is provided along with its priority. The filters in the array parameter can be given in any order. The server will sort them appropriately, putting high priority filters above those with lower priorities. Filters of equal priority will maintain the given order.

### Examples

The following example is taken from a filters test case and illustrates how response filter priorities operate and how response filters can modify outgoing headers and body data.

```swift
struct Filter1: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		response.setHeader(.custom(name: "X-Custom"), value: "Value")
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
}
struct Filter2: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		var b = response.bodyBytes
		b = b.map { $0 == 65 ? 97 : $0 }
		response.bodyBytes = b
		callback(.continue)
	}
}	
struct Filter3: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		var b = response.bodyBytes
		b = b.map { $0 == 66 ? 98 : $0 }
		response.bodyBytes = b
		callback(.done)
	}
}	
struct Filter4: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		XCTAssert(false, "This should not execute")
		callback(.done)
	}
}
	
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
	request, response in
	response.addHeader(.contentType, value: "text/plain")
	response.isStreaming = true
	response.setBody(string: "ABZ")
	response.push {
		_ in
		response.setBody(string: "ABZ")
		response.completed()
	}
})
	
let responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [
	(Filter1(), HTTPFilterPriority.high),
	(Filter2(), HTTPFilterPriority.medium),
	(Filter3(), HTTPFilterPriority.low),
	(Filter4(), HTTPFilterPriority.low)
]
	
let server = HTTPServer()
server.setResponseFilters(responseFilters)
server.serverPort = port
server.addRoutes(routes)
try server.start()
```

The example filters will add a X-Custom header and lowercase any A or B character in the body data. Note that the handler in this example sets the response to streaming mode, meaning that chunked encoding is utilized and the body data is sent out in two descrete chunks.

#### 404 Response Filter

A more useful example follow. This code will create and install a filter which monitors 404 not found responzses and provides a custom message when it finds one.

```swift
struct Filter404: HTTPResponseFilter {
    func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
        callback(.continue)
    }

    func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
        if case .notFound = response.status {
            response.setBody(string: "The file \(response.request.path) was not found.")
            response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
            callback(.done)
        } else {
            callback(.continue)
        }
    }
}

let server = HTTPServer()
server.setResponseFilters([(Filter404(), .high)])
server.serverPort = 8181
try server.start()
```

