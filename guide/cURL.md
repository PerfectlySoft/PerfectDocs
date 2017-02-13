# Network Requests with cURL

Perfect provides a complete interface to the open source cURL library through the use of the cURL free and open command-line tool to transfer files on several supported protocols.

cURL transfers data with URL syntax, and it supports the following protocols:

DICT, FILE, FTP, FTPS, Gopher, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMTP, SMTPS, Telnet, TFTP

cURL has built-in support for SSL certificates, HTTP POST, HTTP PUT, FTP uploading, proxies, cookies, user+password authentication, proxy tunnelling, and more.

### Getting Started

In addition to PerfectLib, you will need the Perfect-cURL dependency in the Package.swift file:

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-Curl.git", 
	majorVersion: 2, minor: 0
	)
```

In each Swift source file that references the cURL classes, include the import directive:

``` swift
import PerfectCURL
```

### Linux Build Notes

Ensure that you have installed libcurl.

```
sudo apt-get install libcurl-dev
```

## Using Perfect cURL

To initialize a cURL operation:

``` swift
let curlObject = CURL(url: "http://www.perfect.org")
```

The current URL property of a cURL object can also be retrieved using the `.url` method:

``` swift
print(curlObject.url)
>> http://www.perfect.org
```

### Setting cURL Options

Setting options for the cURL object is an important part of the process. Many options are available and listing them all is beyond the scope of this document. For a complete list of available cURL options, see the [Perfect API Reference](https://perfect.org/docs/api.html) or the [libcurl API reference](https://curl.haxx.se/libcurl/c/).

To set an option, use the appropriate `.setOption` variant as below:

Where the type of the value is an Int or Int64:

``` swift
curlObject.setOption(<OPTION>, int: <INT>)
```

Where the form of the value is a pointer option value:

``` swift
curlObject.setOption(<OPTION>, v: <UnsafeMutablePointer<Void>>)
```
> Note that the pointer value is not copied or otherwise manipulated or saved. It is up to the caller to ensure the pointer value has a lifetime which corresponds to its usage.

Where a callback function option value is to be set:

``` swift
curlObject.setOption(<OPTION>, f: <curl_func>)
```

Where a string option value is to be set:

``` swift
curlObject.setOption(<OPTION>, s: <String>)
```

For example, where the SSL Peer Verification is to not be performed, the following would be used:

``` swift
curlObject.setOption(CURLOPT_SSL_VERIFYPEER, int: 0)
```

### Performing the cURL Action and Returning Data

Once all the options have been set for the cURL object, the operation can be "performed" in one of the following ways:

This executes the current request:

``` swift
var perf = curlObject.perform()
```

Returning a tuple consisting of: 

* `Bool` - should perform() be called again
* `Int` - the result code
* `[UInt8]` - the header bytes if any
* `[UInt8]` - the body bytes if any

To perform the cURL request in a non-blocking manner, the closure will be called with the resulting code, header, and body data:

``` swift
var perf = curlObject.perform { <Int>, <[UInt8]>, <[UInt8]> }
```

For example, to perform and read the data using a closure:

``` swift
curlObject.perform {
	code, header, body in
	
	print("Request error code \(code)")
	print("Response: \(curlObject.responseCode)")
	print("Header: \(header)")
	print("Body: \(body)")
}

```

When it is necessary to execute the cURL request, and block the current thread until it completes, use the `.performFully` method:

``` swift
var perf = curlObject.performFully()
```

Returning a tuple consisting of: 

* `Int` - the result code
* `[UInt8]` - the header bytes if any
* `[UInt8]` - the body bytes if any


### Reading cURL Response Data Where Result is a Tuple

The parts of the cURL response can be read as follows when a tuple is returned:

``` swift
let curlObject = CURL(url: "http://www.perfect.org")

curlObject.setOption(CURLOPT_SSL_VERIFYPEER, int: 0)

var header = [UInt8]()
var body = [UInt8]()

var perf = curlObject.perform()
while perf.0 {
	if let h = perf.2 {
		header.append(contentsOf: h)
	}
	if let b = perf.3 {
		body.append(contentsOf: b)
	}
	perf = curlObject.perform()
}
if let h = perf.2 {
	header.append(contentsOf: h)
}
if let b = perf.3 {
	body.append(contentsOf: b)
}
let perf1 = perf.1

let response = curlObject.responseCode
print(response == 200, "\(response)")

print("Header size: \(header.count)")
print("Body size: \(body.count)")
```

### cURL Info and Error Codes

`.getInfo` returns the string value for the given `CURLINFO`:

``` swift
let (s:String, c:CURLCode) = curlObject.getInfo(<CURLINFO>)
let (i:Int, c:CURLCode) = curlObject.getInfo(<CURLINFO>)
```

A full list and description of all `CURLINFO` options can be found here: [https://curl.haxx.se/libcurl/c/easy_getinfo_options.html](https://curl.haxx.se/libcurl/c/easy_getinfo_options.html)

To retrieve a string message for the given cURL result code, use `.strError`:

``` swift
let errMsg = curlObject.strError(code: <CURLcode>)
```

### Resetting and Closing the cURL Object

To clean up and reset the cURL object for re-use, use the `.reset()` method:

``` swift
curlObject.reset()
```

If the cURL object created is no longer required, use the `.close()` method instead:

``` swift
curlObject.close()
```
