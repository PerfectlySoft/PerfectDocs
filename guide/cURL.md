# Network Requests with Perfect-CURL

The Perfect-CURL package provides support for [curl](https://curl.haxx.se) in Swift. This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project.

## Building

Ensure you have installed and activated the latest Swift 4.0+ tool chain.

Add this package as a dependency in your Package.swift file.

Swift 4 format:
```
.package(url: "https://github.com/PerfectlySoft/Perfect-CURL.git", from: "3.0.0")
```

Swift 4 format:
```
.Package(url: "https://github.com/PerfectlySoft/Perfect-CURL.git", majorVersion: 3)
```

### Linux Build Notes

Ensure that you have installed libcurl.

```
sudo apt-get install libcurl4-openssl-dev
```

## Usage

`import PerfectCURL`

This package uses a simple request/response model to access URL contents. Start by creating a `CURLRequest` object and configure it according to your needs, then ask it to perform the request and return a response. Responses are represented by `CURLResponse` objects.

Requests can be executed either synchronously - blocking the calling thread until the request completes, asynchronously - delivering the response to a callback, or through a Promise object - performing the request on a background thread giving you a means to chain additional tasks, poll, or wait for completion.

### Creating Requests

`CURLRequest` objects can be created with a URL and a series of options, or they can be created blank and then fully configured. `CURLRequest` provides the following initializers:

```swift
open class CURLRequest {
	// Init with a url and options array.
	public convenience init(_ url: String, options: [Option] = [])
	/// Init with url and one or more options.
	public convenience init(_ url: String, _ option1: Option, _ options: Option...)
	/// Init with array of options.
	public init(options: [Option] = [])
}
```

Options can be provided using either `Array<Option>` or variadic parameters. Options can also be directly added to the `CURLRequest.options` property before the request is executed.

### Configuring Requests

`CURLRequest` options are represented by the `CURLRequest.Option` enum. Each enum case will have zero or more associated values which indicate the parameters for the particular option. For example, the URL for the request could be indicated with the option `.url("https://httpbin.org/post")`. Most of the options that curl makes available are represented in the `CURLRequest.Option` enum. The full list of available options is presented near the end of this document.

#### POST Data

POST field data, including file uploads, are added to request the same way other options are. The `.postField(POSTField)`, `.postData([UInt8])`, and `.postString(String)` enum cases will set the request's POST content. The `.postField` case can be added to a request multiple times as each instance represents one set of name/value pair. The `.postData` and `.postString` cases should be considered mutually exclusive to other post cases as adding either will overwrite any previously set POST content. Adding POST content data of any sort will automatically set the HTTP method to POST.

The `CURLRequest.POSTField` struct is defined as follows:

```swift
open class CURLRequest {
	public struct POSTField {
		/// Init with a name, value and optional mime-type.
		public init(name: String, value: String, mimeType: String? = nil)
		/// Init with a name, file path and optional mime-type.
		public init(name: String, filePath: String, mimeType: String? = nil)
	}
}
```

The example below creates a POST request and adds several name/value pairs as well as a file. The executed request will automatically have a "multipart/form-data" content type.

```swift
let json = try CURLRequest(url, .failOnError,
			               .postField(.init(name: "key1", value: "value1")),
			               .postField(.init(name: "key2", value: "value2")),
			               .postField(.init(name: "file1", filePath: testFile.path, mimeType: "text/plain")))
										.perform().bodyJSON
```

### Fetching Responses

To perform a request, call one of the `CURLRequest.perform` or `CURLRequest.promise` functions. If the request is successful then you will be provided a `CURLResponse` object which can be used to get response data. If the request fails then a `CURLResponse.Error` will be thrown. A request may fail if it is unable to connect, times out, receives a malformed response, or receives a HTTP response with a status code equal to or greater than 400 when the `.failOnError` option is given. If the `.failOnError` option is not given then any valid HTTP response will be a success, regardless of the response status code.

The three functions for obtaining a response are as follows:

```swift
public extension CURLRequest {
	/// Execute the request synchronously. 
	/// Returns the response or throws an Error.
	func perform() throws -> CURLResponse
	/// Execute the request asynchronously.
	/// The parameter passed to the completion callback must be called to obtain the response or throw an Error.
	func perform(_ completion: @escaping (CURLResponse.Confirmation) -> ())
	/// Execute the request asynchronously. 
	/// Returns a Promise object which can be used to monitor the operation.
	func promise() -> Promise<CURLResponse>
}
```

The first `CURLRequest.perform` function executes the request synchronously on the calling thread. The function call will block until the request succeeds or fails. On failure, a `CURLResponse.Error` will be thrown.

The second `CURLRequest.perform` function executes the request asynchronously on background threads as necessary. The parameter passed to this function is a callback which will be given a `CURLResponse.Confirmation` once the request completes or fails. Calling the confirmation parameter from within your callback will either return the `CURLResponse` or throw a `CURLResponse.Error`. 

The third function, `CURLRequest.perform`, will return a `Promise<CURLResponse>` object which can be used to chain further activities and poll or wait for the request to complete. As with the other response generating functions, a `CURLResponse.Error` will be thrown if an error occurs. Information on the Promise object in general can be found in the [Perfect-Thread](http://www.perfect.org/docs/thread.html) documentation.

The following three example shows how each of the functions are used. Each will perform a request and convert the resulting response body from JSON into a [String:Any] dictionary.

• Synchronously fetch an API endpoint and decode it from JSON:

```swift
let url = "https://httpbin.org/get?p1=v1&p2=v2"
let json: [String:Any] = try CURLRequest(url).perform().bodyJSON
```

• Asynchronously fetch an API endpoint and decode it from JSON:

```swift
let url = "https://httpbin.org/post"
CURLRequest(url).perform {
	confirmation in
	do {
		let response = try confirmation()
		let json: [String:Any] = response.bodyJSON
		
	} catch let error as CURLResponse.Error {
		print("Failed: response code \(error.response.responseCode)")
	} catch {
		print("Fatal error \(error)")
	}
}
```

• Asynchronously fetch an API endpoint using a Promise and decode it from JSON:

```swift
let url = "https://httpbin.org/get?p1=v1&p2=v2"
if let json = try CURLRequest(url).promise().then { return try $0().bodyJSON }.wait() {
	...
}
```

The three available functions ranked according to efficiency would be ordered as:

1. Asynchronous `perform`
2. Asynchronous `promise`
3. Synchronous `perform`

When performing CURL requests on a high-traffic server it is advised that one of the asynchronous response functions be used.

#### Reset Request

A `CURLRequest` object can be reused for subsequent connections by calling the `.reset` function. Resetting a request will clear out any previously set options, including the target URL. The `.reset` function accepts as an optional parameter new options with which the request will be reconfigured.

The reset declaration follows:

```swift
public extension CURLRequest {
	/// Reset the request. Clears all options so that the object can be reused.
	/// New options can be provided.
	func reset(_ options: [Option] = [])
	/// Reset the request. Clears all options so that the object can be reused.
	/// New options can be provided.
	func reset(_ option: Option, _ options: Option...)
}
```

Resetting a request will invalidate any previously executed `CURLResponse` objects. The reconfigured request should be reexecuted to obtain an updated `CURLResponse`.

### Response Data

A `CURLResponse` object provides access to the response's content body as either raw bytes, a String or as a JSON decoded [String:Any] dictionary. In addition, meta-information such as the response HTTP headers and status code can be retrieved.

Response body data is made available through a series of get-only `CURLResponse` properties:

```swift
public extension CURLResponse {
	/// The response's raw content body bytes.
	public var bodyBytes: [UInt8]
	/// Get the response body converted from UTF-8.
	public var bodyString: String
	/// Get the response body decoded from JSON into a [String:Any] dictionary.
	/// Invalid/non-JSON body data will result in an empty dictionary being returned.
	public var bodyJSON: [String:Any]
	/// Get the response body decoded from JSON into a decodable 
	/// Invalid/non-JSON body data will throw errors.
	public func bodyJSON<T: Decodable>(_ type: T.Type) throws -> T 
}
```

The remaining response data can be retrieved by calling one of the `CURLResponse.get` functions and passing in an enum value corresponding to the desired data. The enums indicating these values are separated into three groups, each according to the type of data that would be returned; one of String, Int or Double. The enum types are `CURLResponse.Info.StringValue`, `CURLResponse.Info.IntValue`, and `CURLResponse.Info.DoubleValue`. In addition, `get` functions are provided for directly pulling header values from the response.

```swift
public extension CURLResponse {
	/// Get an response info String value.
	func get(_ stringValue: Info.StringValue) -> String?
	/// Get an response info Int value.
	func get(_ intValue: Info.IntValue) -> Int?
	/// Get an response info Double value.
	func get(_ doubleValue: Info.DoubleValue) -> Double?
	/// Get a response header value. Returns the first found instance or nil.
	func get(_ header: Header.Name) -> String?
	/// Get a response header's values. Returns all found instances.
	func get(all header: Header.Name) -> [String]
}
```

For convenience properties have been added for pulling commonly requested data from a response such as `url` and `responseCode`.

The following examples show how to pull header and other meta-data from the response:

```swift
// get the response code
let code = response.get(.responseCode)
```

```swift
// get the response code using the accessor
let code = response.responseCode
```

```swift
// get the "Last-Modified" header from the response
if let lastMod = response.get(.lastModified) {
	...
}
```

### Failures

When a failure occurs a `CURLResponse.Error` object will be thrown. This object provides the CURL error code which was generated (not that this is different from any HTTP response code and is CURL specific). It also provides access to an error message string, and a CURLResponse object which can be used to further inquire about the resulting error.

Note that, depending on the options which were set on the request, the response object obtained after an error may not have any associated content body data.

`CURLResponse.Error` is defined as follows:

```swift
open class CURLResponse {
	/// An error thrown while retrieving a response.
	public struct Error: Swift.Error {
		/// The curl specific request response code.
		public let code: Int
		/// The string message for the curl response code.
		public let description: String
		/// The response object for this error.
		public let response: CURLResponse
	}
}
```

### CURLRequest.Option List

The following is a list of the numerous `CURLRequest` options which can be set. Each enum case indicates the parameter types for the option. These enum values can be used when creating a new `CURLRequest` object or by adding them to an existing object's `.options` array property.

|CURLRequest.Option enum case|Description|
|---|---|
|.url(String)|The URL for the request.|
|.port(Int)|Override the port for the request.|
|.failOnError|Fail on http error codes >= 400.|
|.userPwd(String)|Colon separated username/password string.|
|.proxy(String)|Proxy server address.|
|.proxyUserPwd(String)|Proxy server username/password combination.|
|.proxyPort(Int)|Port override for the proxy server.|
|.timeout(Int)|Maximum time in seconds for the request to complete. The default timeout is never.|
|.connectTimeout(Int)|Maximum time in seconds for the request connection phase. The default timeout is 300 seconds.|
|.lowSpeedLimit(Int)|The average transfer speed in bytes per second that the transfer should be below during `.lowSpeedLimit` seconds for the request to be too slow and abort.|
|.lowSpeedTime(Int)|The time in seconds that the transfer speed should be below the `.lowSpeedLimit` for the request to be considered too slow and aborted.|
|.range(String)|Range request value as a string in the format "X-Y", where either X or Y may be left out and X and Y are byte indexes|
|.resumeFrom(Int)|The offset in bytes at which the request should start from.|
|.cookie(String)|Set one or more cookies for the request. Should be in the format "name=value". Separate multiple cookies with a semi-colon: "name1=value1; name2=value2".|
|.cookieFile(String)|The name of the file holding cookie data for the request.|
|.cookieJar(String)|The name of the file to which received cookies will be written.|
|.followLocation(Bool)|Indicated that the request should follow redirects. Default is false.|
|.maxRedirects(Int)|Maximum number of redirects the request should follow. Default is unlimited.|
|.maxConnects(Int)|Maximum number of simultaneously open persistent connections that may cached for the request.|
|.autoReferer(Bool)|When enabled, the request will automatically set the Referer: header field in HTTP requests when it follows a Location: redirect|
|.krbLevel(KBRLevel)|Sets the kerberos security level for FTP. Value should be one of the following: .clear, .safe, .confidential or .private.|
|.addHeader(Header.Name, String)|Add a header to the request.|
|.addHeaders([(Header.Name, String)])|Add a series of headers to the request.|
|.replaceHeader(Header.Name, String)|Add or replace a header.|
|.removeHeader(Header.Name)|Remove a default internally added header.|
|.sslCert(String)|Path to the client SSL certificate.|
|.sslCertType(SSLFileType)|Specifies the type for the client SSL certificate. Defaults to `.pem`.|
|.sslKey(String)|Path to client private key file.|
|.sslKeyPwd(String)|Password to be used if the SSL key file is password protected.|
|.sslKeyType(SSLFileType)|Specifies the type for the SSL private key file.|
|.sslVersion(TLSMethod)|Force the request to use a specific version of TLS or SSL.|
|.sslVerifyPeer(Bool)|Indicates whether the request should verify the authenticity of the peer's certificate.|
|.sslVerifyHost(Bool)|Indicates whether the request should verify that the server cert is for the server it is known as.|
|.sslCAFilePath(String)|Path to file holding one or more certificates which will be used to verify the peer.|
|.sslCADirPath(String)|Path to directory holding one or more certificates which will be used to verify the peer.|
|.sslCiphers([String])|Override the list of ciphers to use for the SSL connection.|
|.sslPinnedPublicKey(String)|File path to the pinned public key. When negotiating a TLS or SSL connection, the server sends a certificate indicating its identity. A public key is extracted from this certificate and if it does not exactly match the public key provided to this option, curl will abort the connection before sending or receiving any data.|
|.ftpPreCommands([String])|List of (S)FTP commands to be run before the file transfer.|
|.ftpPostCommands([String])|List of (S)FTP commands to be run after the file transfer.|
|.ftpPort(String)|Specifies the local connection port for active FTP transfers.|
|.ftpResponseTimeout(Int)|The time in seconds that the request will wait for FTP server responses.|
|.sshPublicKey(String)|Path to the public key file used for SSH connections.|
|.sshPrivateKey(String)|Path to the private key file used for SSH connections.|
|.httpMethod(HTTPMethod)|HTTP method to be used for the request.|
|.postField(POSTField)|Adds a single POST field to the request. Generally, multiple POST fields are added for a request.|
|.postData([UInt8])|Raw bytes to be used for a POST request.|
|.postString(String)|Raw string data to be used for a POST request.|
|.mailFrom(String)|Specifies the sender's address when performing an SMTP request.|
|.mailRcpt(String)|Specifies the recipient when performing an SMTP request. Multiple recipients may be specified by using this option multiple times.|

### CURLResponse.Info List

The lists which follow describe the `CURLResponse.Info` cases which are used with the `CURLResponse.get` function to retrieve response information. The lists are grouped according to the type of data which would be returned; `StringValue`, `IntValue`, and `DoubleValue`, respectively. 

|CURLResponse.Info.StringValue enum case|Description|
|---|---|
|.url|The effective URL for the request/response. This is ultimately the URL from which the response data came from. This may differ from the request's URL in the case of a redirect.|
|.ftpEntryPath|The initial path that the request ended up at after logging in to the FTP server.|
|.redirectURL|The URL that the request *would have* been redirected to.|
|.localIP|The local IP address that the request used most recently.|
|.primaryIP|The remote IP address that the request most recently connected to.|
|.contentType|The content type for the request. This is read from the "Content-Type" header.|

|CURLResponse.Info.IntValue enum case|Description|
|---|---|
|.responseCode|The last received HTTP, FTP or SMTP response code.|
|.headerSize|The total size in bytes of all received headers.|
|.requestSize|The total size of the issued request in bytes. This will indicate the cumulative total of all requests sent in the case of a redirect.|
|.sslVerifyResult|The result of the SSL certificate verification.|
|.redirectCount|The total number of redirections that were followed.|
|.httpConnectCode|The last received HTTP proxy response code to a CONNECT request.|
|.osErrno|The OS level errno which may have triggered a failure.|
|.numConnects|The number of connections that the request had to make in order to produce a response.|
|.primaryPort|The remote port that the request most recently connected to|
|.localPort|The local port that the request used most recently|

|CURLResponse.Info.DoubleValue enum case|Description|
|---|---|
|.totalTime|The total time in seconds for the previous request.|
|.nameLookupTime|The total time in seconds from the start until the name resolving was completed.|
|.connectTime|The total time in seconds from the start until the connection to the remote host or proxy was completed.|
|.preTransferTime|The time, in seconds, it took from the start until the file transfer is just about to begin.|
|.sizeUpload|The total number of bytes uploaded.|
|.sizeDownload|The total number of bytes downloaded.|
|.speedDownload|The average download speed measured in bytes/second.|
|.speedUpload|The average upload speed measured in bytes/second.|
|.contentLengthDownload|The content-length of the download. This value is obtained from the Content-Length header field.|
|.contentLengthUpload|The specified size of the upload.|
|.startTransferTime|The time, in seconds, it took from the start of the request until the first byte was received.|
|.redirectTime|The total time, in seconds, it took for all redirection steps include name lookup, connect, pre-transfer and transfer before final transaction was started.|
|.appConnectTime|The time, in seconds, it took from the start until the SSL/SSH connect/handshake to the remote host was completed.|