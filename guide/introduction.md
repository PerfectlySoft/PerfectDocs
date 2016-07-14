# Perfect: Server-Side Swift

[![Swift 3.0](https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat)](https://developer.apple.com/swift/)
[![Platforms OS X | Linux](https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat)](https://developer.apple.com/swift/)
[![License Apache](https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat)](http://perfect.org/licensing.html)
[![Docs](https://img.shields.io/badge/docs-99%25-yellow.svg?style=flat)](http://www.perfect.org/docs/)
[![GitHub issues](https://img.shields.io/github/issues/PerfectlySoft/Perfect.svg)](https://github.com/PerfectlySoft/Perfect/issues)
[![codebeat](https://codebeat.co/badges/85f8f628-6ce8-4818-867c-21b523484ee9)](https://codebeat.co/projects/github-com-perfectlysoft-perfect)
[![Twitter](https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat)](http://twitter.com/PerfectlySoft)
[![Join the chat at https://gitter.im/PerfectlySoft/Perfect](https://img.shields.io/badge/Gitter-Join%20Chat-brightgreen.svg)](https://gitter.im/PerfectlySoft/Perfect?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

**2016-07-07 NOTE:** We have moved the core HTTP, HTTPServer and Mustache code into their own repositories. This introduces compilation issues with existing code and you will need to import one or more of the following packages, depending on your usage:

```swift
import PerfectMustache
import PerfectHTTP
import PerfectHTTPServer
```

You will also need to adjust your Package.swift file. You should only need to change the Perfect dependency to the following:

```
https://github.com/PerfectlySoft/Perfect-HTTPServer.git
```

----------------

**The master branch of this project currently compiles with the default Swift 3.0 toolchain included in Xcode 8 beta 2. On Ubuntu use the *Swift 3.0 Preview 2* toolchain, released July 7th.**

**Important:** On OS X you must set the Xcode command line tools preference as follows:
![Xcode Prefs](http://www.perfect.org/docs/assets/xcode_prefs.png) 

If you do not do this you will experience compile time errors when using SPM on the command line.

If you are still having build problems with any of the code in our repositories, try doing a clean build with Swift Package Manager by typing:

```
swift build --clean=dist;
swift build
```

----------------

Perfect is an application server for Linux or OS X which provides a framework for developing web and other REST services in the Swift programming language. Its primary focus is on facilitating mobile apps which require backend server software, enabling you to use one language for both front and back ends.

Perfect operates using either its own stand-alone HTTP/HTTPS server or through FastCGI. It provides a system for loading your own Swift based modules at startup and for interfacing those modules with its request/response objects or to the built-in mustache template processing system.

Perfect is built on its own high performance completely asynchronous networking engine with the goal of providing a scalable option for internet services. It supports SSL out of the box and provides a suite of tools which are commonly required by internet servers, such as WebSockets and iOS push notifications, but does not limit your options. Feel free to swap in your own favorite JSON or templating systems, etc.