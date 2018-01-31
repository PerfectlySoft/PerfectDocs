# Perfect New Relic Library for Linux 

This project provides a Swift Agent SDK for New Relic.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project but can be used as an independent module.

## Release Note

This project is only compatible with Ubuntu 16.04 and Swift 4.0 Tool Chain.

## Quick Start

Please use [Perfect Assistant](http://www.perfect.org/en/assistant/) to import this project, otherwise an install script is available for Ubuntu 16.04:

```
$ git clone https://github.com/PerfectlySoft/Perfect-NewRelic-linux.git
$ cd Perfect-libNewRelic-linux
$ sudo ./install.sh
```

During the installation, it will automatically ask for **license key**, **application name**, language and its version. Then it will install the command line `newrelic-collector-client-daemon` as a service which you can find the configuration on `/usr/local/etc/newrelic.service`.

Configure Package.swift:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-NewRelic-linux.git", majorVersion: 3)
```

Import library into your code ( **NOTE** Since Swift 4.0 on linux has a significant linker issue, so module `PerfectNewRelic` must be accompanied with `Foundation`):

``` swift
import PerfectNewRelic
import Foundation
```

Aside of Swift - C syntax conversion differences, document can be found on [New Relic Agent SDK](https://docs.newrelic.com/docs/agents/agent-sdk/using-agent-sdk/using-agent-sdk)

## Configuration

Post Installation & Configuration can be found on [New Relic - Configuring the Agent SDK](https://docs.newrelic.com/docs/agents/agent-sdk/installation-configuration/configuring-agent-sdk), please note that NewRelic instance is configured as **Daemon Mode** which is highly recommended because the *Embedded-mode* is still experimental.

If success, you can create NewRelic instance easily:

``` swift
let nr = try NewRelic()
```

- Create a function to receive status change notifications:

``` swift
nr.registerStatus { code in
	guard let status = NewRelic.Status(rawValue: code) else {
		// something wrong here
	}//end guard
	switch status {
		case .STARTING: // New Relic Daemon Service is starting
		case .STARTED: // New Relic Daemon Service is started
		case .STOPPING: // New Relic Daemon Service is stopping
		default: // shutdown already
   }//end case
}//end callback
```
## Limiting or disabling Agent SDK settings

According to [New Relic Limiting or disabling Agent SDK Settings](https://docs.newrelic.com/docs/agents/agent-sdk/installation-configuration/limiting-or-disabling-agent-sdk-settings), the following settings are available in Perfect NewRelic:

If you want to ... | Use this setting ...
-------------------|---------------------
Disable data collection during a transaction|`nr.enableInstrumentation(false)`
Configure the number of trace segments collected in a transaction trace|`let t = try Transaction(nr, maxTraceSegments: 50)` // // Only collect up to 50 trace segments

## API Quick Help

Base on [New Relic's Document of Using the Agent SDK](https://docs.newrelic.com/docs/agents/agent-sdk/using-agent-sdk/using-agent-sdk), Perfect NewRelic library provides identically the same functions of New Relic Agent SDK in Swift:

### Instruments & Profiling

Function | `recordMetric()`
---|---
Demo|`try nr.recordMetric(name: "ActiveUsers", value: 25)`
Description|Record a custom metric.
Parameters| - name: name of the metric. <br> - value: value of the metric.

Function | `recordCPU()`
---|---
Demo|`try nr.recordCPU(timeSeconds: 5.0, usagePercent: 1.2)`
Description|Record CPU user time in seconds and as a percentage of CPU capacity.
Parameters| - timeSeconds: Double, number of seconds CPU spent processing user-level code <br> - usagePercent: Double, CPU user time as a percentage of CPU capacity

Function | `recordMemory()`
---|---
Demo|`try nr.recordMemory(megabytes: 32)`
Description|Record the current amount of memory being used.
Parameters| - megabytes: Double, amount of memory currently being used

### Transaction

Transaction in Perfect NewRelic has been defined as a class, with constructior as below:

``` swift
public init(_ instance: NewRelic,
    webType: Bool? = nil,
    category: String? = nil,
    name: String? = nil,
    url: String? = nil,
    attributes: [String: String],
    maxTraceSegments: Int? = nil
  ) throws
```

#### Constructor parameters:

- instance: NewRelic instance, **required**.
- webType: **optional**. true for WebTransaction and false for other. default is true.
- category: **optional**. name of the transaction category, default is 'Uri'
- name: **optional**. transaction name
- url: **optional**. request url for a web transaction
- attributes: **optional**. transaction attributes, pair of "name: value"
- maxTraceSegments: **optional**. Set the maximum number of trace segments allowed in a transaction trace. By default, the maximum is set to 2000, which means the first 2000 segments in a transaction will create trace segments if the transaction exceeds the trace threshold (4 x apdex_t).

#### Demo of Transaction Class Initialization:

``` swift
let nr = NewRelic()
let t = try Transaction(nr, webType: false,
	category: "my-class-1", name: "my-transaction-name",
	url: "http://localhost",
	attributes: ["tom": "jerry", "pros":"cons", "muddy":"puddels"],
	maxTraceSegments: 2000)
```

#### Error Notice

Perfect NewRelic provides `setErrorNotice()` function for transactions:

``` swift
try t.setErrorNotice(
	exceptionType: "my-panic-type-1",
	errorMessage: "my-notice",
	stackTrace: "my-stack",
	stackFrameDelimiter: "<frame>")
```

Parameters of `setErrorNotice()`:

- exceptionType: type of exception that occurred
- errorMessage: error message
- stackTrace: print stack trace when error occurred
- stackFrameDelimiter:  delimiter to split stack trace into frames

#### Segments

Segments in a transaction can be either Generic, DataStore or External, see demo below:

``` swift
// assume t is a transaction
try t.setErrorNotice(exceptionType: "my-panic-type-1", errorMessage: "my-notice", stackTrace: "my-stack", stackFrameDelimiter: "<frame>")

let root = try t.segBeginGeneric(name: "my-segment")
// do some generic operations in this transaction
try t.segEnd(root)

// NOTE: it will automatically obfuscate the sql input and rollup if failed as well
let sub = try t.segBeginDataStore(table: "my-table", operation: .INSERT, sql: "INSERT INTO table(field) value('000-000-0000')")
// do some data operations in this transaction
try t.segEnd(sub)

let s2 = try t.segBeginExternal(host: "perfect.org", name: "my-seg")
// do some external operations in this transaction
try t.segEnd(s2)
```

Parameters:

- parentSegmentId: id of parent segment, root segment by default, i.e., ` NewRelic.ROOT_SEGMENT`.
- name: name to represent segment

