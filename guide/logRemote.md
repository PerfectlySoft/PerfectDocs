# Remote Logging

Using the `PerfectLogger` module, events can be logged to a specified remote Perfect Log Server, in addition to the console.

The [Perfect Log Server](https://github.com/PerfectServers/Perfect-LogServer) is a stand-alone project that can be deployed on your own servers.


## Using in your project

To include the dependency in your project, add the following to your project's Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Logger.git", majorVersion: 3),
```

Now add the import directive to the file you wish to use the logging in:

``` swift 
import PerfectLogger
```

## Configuration
Three configuration parameters are required:

``` swift
// Your token
RemoteLogger.token = "<your token>"

// App ID (Optional)
RemoteLogger.appid = "<your appid>"

// URL to access the log server. 
// Note, this is not the full API path, just the host and port.
RemoteLogger.logServer = "http://localhost:8181"

```

## Usage

To log events to the log server:

``` swift
var obj = [String: Any]()
obj["one"] = "donkey"
RemoteLogger.critical(obj)
```

## Linking events with "eventid"

Each log event returns an event id string. If an eventid string is supplied to the directive then it will use the supplied eventid in the log directive instead. This makes it easy to link together related events.

``` swift
let eid = RemoteLogger.critical(obj)
RemoteLogger.info(obj, eventid: eid)
```

The returned eventid is marked `@discardableResult` and therefore can be safely ignored if not required for re-use.
