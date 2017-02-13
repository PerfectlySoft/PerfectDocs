# HTTPResponse

When handling a request, all client interaction is performed through the provided HTTPRequest and HTTPResponse objects. 

The HTTPResponse object contains all outgoing response data. It consists of the HTTP status code and message, the HTTP headers, and any response body data. HTTPResponse also contains the ability to stream or push chunks of response data to the client, and to complete or terminate the request.

In all of the sections below, unless otherwise noted, the properties and functions are part of the HTTPResponse protocol.

### Relevant Examples

* [Perfect-Cookie-Demo](https://github.com/PerfectExamples/Perfect-Cookie-Demo)
* [Perfect-HTTPRequestLogging](https://github.com/PerfectExamples/Perfect-HTTPRequestLogging)

### HTTP Status

The HTTP status indicates to the client whether or not the request was successful, if there was an error, or if it should take any other action. By default, the HTTPResponse object contains a 200 OK status. The status can be set to any other value if needed. HTTP status codes are represented by the ```HTTPResponseStatus``` enumeration (enum). This enum contains a case for each of the official status codes as well as a ```.custom(code: Int, message: String)``` case.

The response status is set with the following property:

```swift
/// The HTTP response status.
var status: HTTPResponseStatus { get set }
```

### Response Headers

Response headers can be retrieved, set, or iterated. Official/common header names are represented by the ```HTTPResponseHeader.Name``` enum. This also contains a case for custom header names: ```.custom(name: String)```.

```swift
/// Returns the requested outgoing header value.
func header(_ named: HTTPResponseHeader.Name) -> String?
/// Add a header to the outgoing response.
/// No check for duplicate or repeated headers will be made.
func addHeader(_ named: HTTPResponseHeader.Name, value: String)
/// Set the indicated header value. 
/// If the header already exists then the existing value will be replaced.
func setHeader(_ named: HTTPResponseHeader.Name, value: String)
/// Provide access to all current header values.
var headers: AnyIterator<(HTTPResponseHeader.Name, String)> { get }
```

HTTPResponse provides higher level support for setting HTTP cookies. This is accomplished by creating a cookie object and adding it to the response object.

```swift
/// This bundles together the values which will be used to set a cookie in the outgoing response
public struct HTTPCookie {
	/// Cookie public initializer
	public init(name: String,
	            value: String,
	            domain: String?,
	            expires: Expiration?,
	            path: String?,
	            secure: Bool?,
	            httpOnly: Bool?)
}
```

Cookies are added to the HTTPResponse with the following function:

```swift
/// Add a cookie to the outgoing response.
func addCookie(_ cookie: HTTPCookie)
```

When a cookie is added it is properly formatted and added as a "Set-Cookie" header.

### Body Data

The response's current body data is exposed through the following property:

```swift
/// Body data waiting to be sent to the client.
/// This will be emptied after each chunk is sent.
var bodyBytes: [UInt8] { get set }
```

Data can be either directly added to this array, or can be added through one of the following convenience functions. These functions either set completely or append to the body bytes using either raw UInt8 bytes or String data. String data will be converted to UTF-8. The last function permits a [String:Any] dictionary to be converted into a JSON string.

```swift
/// Append data to the bodyBytes member.
func appendBody(bytes: [UInt8])
/// Append String data to the outgoing response.
/// All such data will be converted to a UTF-8 encoded [UInt8]
func appendBody(string: String)
/// Set the bodyBytes member, clearing out any existing data.
func setBody(bytes: [UInt8])
/// Set the String data of the outgoing response, clearing out any existing data.
/// All such data will be converted to a UTF-8 encoded [UInt8]
func setBody(string: String)
/// Encodes the Dictionary as a JSON string and converts that to a UTF-8 encoded [UInt8]
func setBody(json: [String:Any]) throws
```

When responding to a client, it is vital that a content-length header be included. When the HTTPResponse object begins sending the accumulated data to the client, it will check to see if a content length has been set. If it has not, the header will be set based on the count of bytes in the ```bodyBytes``` array.

The following function will push all current response headers and any body data:

```swift
/// Push all currently available headers and body data to the client.
/// May be called multiple times.
func push(callback: (Bool) -> ())
```

During most common requests, it is not necessary to call this method as the system will automatically flush all pending outgoing data when the request is completed. However, in some circumstances it may be desirable to have more direct control over this process. 

For example, if one were to serve a very large file, it may not be practical to read the entire file content into memory and set the body bytes. Instead, one would set the response's content length to the size of the file and then, in chunks, read some of the file data, set the body bytes, and then call the ```push``` function repeatedly until all of the file has been sent. The ```push``` function will call the provided callback function parameter with a Boolean. This bool will be true if the content was successfully sent to the client. If the bool is false, then the request should be considered to have failed, and no further action should be taken.

### Streaming

In some cases, the content length of the response cannot be easily determined. For example, if one were streaming live video or audio content, then it may not be possible to set a content-length header. In such cases, set the HTTPResponse object into streaming mode. When using streaming mode, the response will be sent as HTTP-chunked encoding. When streaming, it is not required that the content length be set. Instead, add content to the body as usual, and call the ```push``` function. If the push succeeds, then the body data will be empty and ready for more data to be added. Continue calling ```push``` until the request has ended, or until push gives a ```false``` to the callback.

```swift
/// Indicate that the response should attempt to stream all outgoing data.
/// This is primarily used when the resulting content length can not be known.
var isStreaming: Bool { get set }
```

If streaming is to be used, it is required that ```isStreaming``` be set to true before any data is pushed to the client.

### Request Completion

**Important**: When a request has completed, it is **required** that the HTTPResponse's ```completed``` function be called. This will ensure that all pending data is delivered to the client, and the underlying TCP connection will be either closed, or in the case of HTTP keep-alive, a new request will be read and processed.

```swift
/// Indicate that the request has completed.
/// Any currently available headers and body data will be pushed to the client.
/// No further request related activities should be performed after calling this.
func completed()
```
