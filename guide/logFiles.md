# File Logging

Using the `PerfectLogger` module, events can be logged to a specified file, in addition to the console.

## Usage

Add the dependency to your project's Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Logger.git", majorVersion: 3),
```

Now add the `import` directive to the file you wish to use the logging in:

``` swift
import PerfectLogger
```

To log events to the local console as well as a file:

``` swift
LogFile.debug("debug message", logFile: "test.txt")
LogFile.info("info message", logFile: "test.txt")
LogFile.warning("warning message", logFile: "test.txt")
LogFile.error("error message", logFile: "test.txt")
LogFile.critical("critical message", logFile: "test.txt")
LogFile.terminal("terminal message", logFile: "test.txt")
```

To log to the default file, omit the file name parameter.

## Linking events with "eventid"

Each log event returns an event id string. If an eventid string is supplied to the directive then it will use the supplied eventid in the log file instead. This makes it easy to link together related events.

``` swift
let eid = LogFile.warning("test 1")
LogFile.critical("test 2", eventid: eid)
```

returns:

```
[WARNING] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] test 1
[CRITICAL] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] test 2
```

The returned eventid is marked `@discardableResult` and therefore can be safely ignored if not required for re-use.


## Setting a custom Logfile location

The default logfile location is `./log.log`. To set a custom logfile location, set the `LogFile.location` variable:

``` swift
LogFile.location = "/var/log/myLog.log"
```

Messages can now be logged directly to the file as set by using:

``` swift
LogFile.debug("debug message")
LogFile.info("info message")
LogFile.warning("warning message")
LogFile.error("error message")
LogFile.critical("critical message")
LogFile.terminal("terminal message")
```

## Sample output

```
[DEBUG] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] a debug message
[INFO] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] an informational message
[WARNING] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] a warning message
[ERROR] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] an error message
[CRITICAL] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] a critical message
[EMERG] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] an emergency message
```
