# HTTP Request Logging

To log HTTP requests to a file, use the `Perfect-RequestLogger` module.

## Relevant Examples

* [Perfect-HTTPRequestLogging](https://github.com/PerfectExamples/Perfect-HTTPRequestLogging)
* [Perfect-Session-Memory-Demo](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)

## Usage

Add the following dependency to the `Package.swift` file:

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-RequestLogger.git", majorVersion: 3)
```

In each file you wish to implement logging, import the module:

``` swift 
import PerfectRequestLogger
```

## When using PerfectHTTP 2.1 or later

Add to `main.swift` after instantiating your `server`:

```swift
// Instantiate a logger
let httplogger = RequestLogger()

// Configure Server
var confData: [String:[[String:Any]]] = [
	"servers": [
		[
			"name":"localhost",
			"port":8181,
			"routes":[],
			"filters":[
				[
					"type":"response",
					"priority":"high",
					"name":PerfectHTTPServer.HTTPFilter.contentCompression,
					],
				[
					"type":"request",
					"priority":"high",
					"name":RequestLogger.filterAPIRequest,
					],
				[
					"type":"response",
					"priority":"low",
					"name":RequestLogger.filterAPIResponse,
					]
			]
		]
	]
]
```
The important parts of the configuration spec to add for enabling the Request Logger are:

``` swift
[
	"type":"request",
	"priority":"high",
	"name":RequestLogger.filterAPIRequest,
],
[
	"type":"response",
	"priority":"low",
	"name":RequestLogger.filterAPIResponse,
]
```
These request & response filters add the required hooks to mark the beginning and the completion of the HTTP request and response.



## When using PerfectHTTP 2.0

Add to `main.swift` after instantiating your `server`:

```swift
// Instantiate a logger
let httplogger = RequestLogger()

// Add the filters
// Request filter at high priority to be executed first
server.setRequestFilters([(httplogger, .high)])
// Response filter at low priority to be executed last
server.setResponseFilters([(httplogger, .low)])
```

These request & response filters add the required hooks to mark the beginning and the completion of the HTTP request and response.


## Setting a custom Logfile location

The default logfile location is `/var/log/perfectLog.log`. To set a custom logfile location, set the `RequestLogFile.location` property:

``` swift
RequestLogFile.location = "/var/log/myLog.log"
```

## Example Log Output

```
[INFO] [62f940aa-f204-43ed-9934-166896eda21c] [servername/WuAyNIIU-1] 2016-10-07 21:49:04 +0000 "GET /one HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000436007976531982s
[INFO] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [servername/WuAyNIIU-2] 2016-10-07 21:49:06 +0000 "GET /two HTTP/1.1" from 127.0.0.1 - 200 64B in 0.000207006931304932s
```

This module expands on earlier work by [David Fleming](https://github.com/dabfleming).
