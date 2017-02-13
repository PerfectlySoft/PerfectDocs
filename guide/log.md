# Log

Perfect has a built-in error logging system that allows messages to be logged at several different levels. Each log level can be routed to either the console or the system log.

The built-in log levels, in order of increasing severity:

* **debug:** Log lines are preceded by `[DBG]`
* **info:** Log lines are preceded by `[INFO]`
* **warning:** Log lines are preceded by `[WARN]`
* **error:** Log lines are preceded by `[ERR]`
* **critical:** Log lines are preceded by `[CRIT]`
* **terminal:** Log lines are preceded by `[TERM]`

### To Log Information to the Console:

``` swift
Log.debug(message: "Line 123: value \(myVar)")
Log.info(message: "At Line 123")
Log.warning(message: "Entered error handler")
Log.error(message: "Error condition: \(errorMessage)")
Log.critical(message: "Exception Caught: \(exceptionVar)")
Log.terminal(message: "Uncaught exception, terminating. \(infoVar)")
```

### To Log Information to the System Log:

If you wish to pipe all log entries to the system log, set the `Log.logger` property to `SysLogger()` early in the application setup. Once this has been executed, all output will be logged to the System Log file, and echoed to the console.

``` swift
Log.logger = SysLogger()
```

If you wish to change the logger process back to only the console at any point, set the property back to `ConsoleLogger()`

``` swift
Log.logger = ConsoleLogger()
```
