# SysProcess

Perfect provides the ability to execute local processes or shell commands through the `SysProcess` type. This type allows local processes to be launched with an array of parameters and shell variables. Some processes will execute and return a result immediately. Other processes can be left open for interactive read/write operations.

## Relevant Examples

* [Perfect-System](https://github.com/PerfectExamples/Perfect-System)

## Setup

Add the "Perfect" project as a dependency in your Package.swift file:

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/PerfectLib.git",
	majorVersion: 3
	)
```
In your file where you wish to use SysProcess, import the PerfectLib and add either SwiftGlibc or Darwin:

``` swift
import PerfectLib

#if os(Linux)
	import SwiftGlibc
#else
	import Darwin
#endif
```

### Executing a SysProcess Command

The following function `runProc` accepts a command, an array of arguments, and optionally outputs the response from the command.

``` swift
func runProc(cmd: String, args: [String], read: Bool = false) throws -> String? {
	let envs = [("PATH", "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin")]
	let proc = try SysProcess(cmd, args: args, env: envs)
	var ret: String?
	if read {
		var ary = [UInt8]()
		while true {
			do {
				guard let s = try proc.stdout?.readSomeBytes(count: 1024) where s.count > 0 else {
					break
				}
				ary.append(contentsOf: s)
			} catch PerfectLib.PerfectError.fileError(let code, _) {
				if code != EINTR {
					break
				}
			}
		}
		ret = UTF8Encoding.encode(bytes: ary)
	}
	let res = try proc.wait(hang: true)
	if res != 0 {
		let s = try proc.stderr?.readString()
		throw  PerfectError.systemError(Int32(res), s!)
	}
	return ret
}

let output = try runProc(cmd: "ls", args: ["-la"], read: true)
print(output)
```

Note that the `SysProcess` command is executed in this example with a hardcoded environment variable.

### SysProcess Members

#### stdin
`stdin` is the standard input file stream.

#### stdout
`stdout` is the standard output file stream.

#### stderr
`stderr` is the standard error file stream.

#### pid
`pid` is the process identifier.

### SysProcess Methods

#### isOpen

Returns true if the process was opened and was running at some point.

Note that the process may not be currently running. Use `wait(false)` to check if the process is currently running.

``` swift
myProcess.isOpen()
```

#### close

`close` terminates the process and cleans up.

``` swift
myProcess.close()
```

#### detach

Detach from the process such that it will not be manually terminated when this object is uninitialized.

``` swift
myProcess.detach()
```

#### wait

Determine if the process has completed running and retrieve its result code.

``` swift
myProcess.wait(hang: <Bool>)
```

#### kill

Terminate the process and return its result code.

``` swift
myProcess.kill(signal: <Int32 = SIGTERM>)
```
Response is an `Int32` result code.
