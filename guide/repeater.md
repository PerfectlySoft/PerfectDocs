# Perfect Repeater

This package provides a method to schedule repeating/recurring events.

## Relevant Examples

* [Perfect-Repeater-Example](https://github.com/PerfectExamples/Perfect-Repeater-Demo)

## Getting Started

In addition to the PerfectLib, you will need the Perfect-Repeater dependency in the Package.swift file:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Repeater.git", majorVersion: 3)
```

## Using Perfect Repeater

Import the Perfect Repeater into each file that you wish to use the functions in: 

``` swift
import PerfectRepeater
```

The base form of executing this is:

``` swift
Repeater.exec(timer: <Double>, callback: <Closure>)
```

The `timer` value is the time in seconds to repeat the event.

The `callback` contains a closure containing code to execute. This must contain a boolean return value. Returning `true` will re-queue the event, and `false` will remove the event from the queue.

The following code demonstrates the process of repeating a closure containing your code and optionally re-queuing:

``` swift
var opt = 1

let c = {
	() -> Bool in
	print("XXXXXX")
	return true
}
let cc = {
	() -> Bool in
	print("Hello, world! (\(opt))")
	if opt < 10 {
		opt += 1
		return true
	} else {
		print("cc exiting.")
		return false
	}
}

Repeater.exec(timer: 3.0, callback: c)
Repeater.exec(timer: 2.0, callback: cc)
```