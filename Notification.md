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
