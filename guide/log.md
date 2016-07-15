# Log

Perfect has a built-in error logging system that allows messages to be logged at several different levels. Each log level can be routed to either console or the system log.

The built-in log levels, in order of increasing severity:

* **debug:** Log lines are preceeded by `[DBG]`
* **info:** Log lines are preceeded by `[INFO]`
* **warning:** Log lines are preceeded by `[WARN]`
* **error:** Log lines are preceeded by `[ERR]`
* **critical:** Log lines are preceeded by `[CRIT]`
* **terminal:** Log lines are preceeded by `[TERM]`

### To log information to the console:

``` swift
Log.warning(message: "Hello, World!")
```

### To log information to the System Log:

If you wish to pipe all log entries to the system log, early in the application startup set the `Log.logger` property to `SysLogger()`. Once this has been executed all output will be logged to the System Log file, and echoed to the console.

``` swift
Log.logger = SysLogger()
```

If you wish to change the logger process back to the console only at any point, set the property back to `ConsoleLogger()`

``` swift
Log.logger = ConsoleLogger()
```
