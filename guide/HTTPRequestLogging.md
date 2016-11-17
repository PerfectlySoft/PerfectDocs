# HTTP Request Logging

To log HTTP requests to a file, use the `Perfect-RequestLogger` module.

## Usage

Add the following dependency to the `Package.swift` file:

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-RequestLogger.git", majorVersion: 0)
```

For each file you wish to directly reference the logging, import the module:

``` swift 
import PerfectRequestLogger
```

Add to `main.swift` after instantiating your `server`:

```swift
// Instantiate a logger
let myLogger = RequestLogger()

// Add the filters
// Request filter at high priority to be executed first
server.setRequestFilters([(myLogger, .high)])
// Response filter at low priority to be executed last
server.setResponseFilters([(myLogger, .low)])
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
