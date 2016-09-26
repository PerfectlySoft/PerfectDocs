# HTTPRequest

When handling a request, all client interaction is performed through HTTPRequest and HTTPResponse objects. The HTTPRequest object makes available all client headers, query parameters, POST body data, and other relevant information such as the client IP address and URL variables.HTTPRequest objects will handle parsing and decoding all "application/x-www-form-urlencoded", as well as "multipart/form-data" content type requests. It will make the data for any other content types available in a raw, unparsed form. When handling multipart form data, HTTPRequest will automatically decode the data and create temporary files for any file uploads contained therein. These files will exist until the request ends after which they will be automatically deleted.In all of the sections below, the properties and functions are part of the HTTPRequest protocol.

### Meta Data

HTTPRequest provides several pieces of data which are not explicitly sent by the client. Information such as the client and server IP addresses, TCP ports, and document root fit into this category.Client and server addresses are made available as tuples (a finite ordered list of elements) containing the IP addresses and respective ports of each:

```swift
/// The IP address and connecting port of the client.
var remoteAddress: (host: String, port: UInt16) { get }
/// The IP address and listening port for the server.
var serverAddress: (host: String, port: UInt16) { get }
```

When a server is created, you can set its canonical name, or CNAME. This can be useful in a variety of circumstances. For example, when creating full links to your server. When a HTTPRequest is created, the server will apply a CNAME to it. It is made available through the following property:

```swift
/// The canonical name for the server.
var serverName: String { get }
```

The server's document root is the directory from which static content is generally served. If you are not serving static content, then this document root may not exist. The document root is configured on the server before it begins accepting requests. Its value is transferred to the HTTPRequest when it is created. When attempting to access static content such as a Mustache template, one would generally prefix all file paths with this document root value.

```swift
/// The server's document root from which static file content will generally be served.
var documentRoot: String { get }
```

### Request Line

An HTTPRequest line consists of a method, path, query parameters, and an HTTP protocol identifier. An example HTTPRequest line may appear as follows:

```
GET /path?q1=v1&q2=v2 HTTP/1.1
```

HTTPRequest makes the parsed request line available through the properties below. The query parameters are presented as an array of name/value tuples in which all names and values have been URL decoded:

```swift
/// The HTTP request method.
var method: HTTPMethod { get set }
/// The request path.
var path: String { get set }
/// The parsed and decoded query/search arguments.
var queryParams: [(String, String)] { get }
/// The HTTP protocol version. For example (1, 0), (1, 1), (2, 0)
var protocolVersion: (Int, Int) { get }
```

During the routing process, the route URI may have consisted of URL variables, and these will have been parsed and made available as a dictionary:

```swift
/// Any URL variables acquired during routing the path to the request handler.
var urlVariables: [String:String] { get set }
```

An HTTPRequest also makes the full request URI available. It will include the request path as well as any URL encoded query parameters:

```swift
/// Returns the full request URI.
var uri: String { get }
```

### Client Headers

Client request headers are made available either keyed by name or through an iterator permitting all header names and values to be accessed. HTTPRequest will automatically parse and make available all HTTP cookie names and values. All possible request header names are represented in the enumeration type ```HTTPRequestHeader.Name```, which also includes a ```.custom(name: String)``` case for unaccounted header names.

It is possible to set client headers after the request has been read. This would be useful in, for example, HTTPRequest filters as they may need to rewrite or add certain headers:

```swift
/// Returns the requested incoming header value.
func header(_ named: HTTPRequestHeader.Name) -> String?
/// Add a header to the response.
/// No check for duplicate or repeated headers will be made.
func addHeader(_ named: HTTPRequestHeader.Name, value: String)
/// Set the indicated header value.
/// If the header already exists then the existing value will be replaced.
func setHeader(_ named: HTTPRequestHeader.Name, value: String)
/// Provide access to all current header values.
var headers: AnyIterator<(HTTPRequestHeader.Name, String)> { get }
```

Cookies are made available through an array of name/value tuples.

```swift
/// Returns all the cookie name/value pairs parsed from the request.
var cookies: [(String, String)] 
```

### GET and POST Parameters

For a detailed discussion of accessing GET and POST paramaters, please see [Using Form Data](formData.md).

### Body Data

For the content types "application/x-www-form-urlencoded" and "multipart/form-data", HTTPRequest will automatically parse and make the values available through the ```postParams``` or ```postFileUploads``` properties, respectively. 

For a more detailed discussion of file upload handling, please see [File Uploads](fileUploads.md). For more details on ```postParams```, please see [Using Form Data](formData.md).

Request body data with other content types are not parsed and are made available either as raw bytes or as String data. For example, for a client submitting JSON data, one would want to access the body data as a String which would then be decoded into a useful value.HTTPRequest makes body data available through the following properties:

```swift
/// POST body data as raw bytes.
/// If the POST content type is multipart/form-data then this will be nil.
var postBodyBytes: [UInt8]? { get set }
/// POST body data treated as UTF-8 bytes and decoded into a String, if possible.
/// If the POST content type is multipart/form-data then this will be nil.
var postBodyString: String? { get }
```

It's important to note that if the request has the "multipart/form-data" content type, then the ```postBodyBytes``` property will be nil. Otherwise, it will always contain the request body data regardless of the content type.

The ```postBodyString``` property will attempt to convert the body data from UTF-8 into a String. It will return nil if there is no body data or if the data could not successfully be converted from UTF-8.