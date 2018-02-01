# Perfect INI File Parser 

This project provides an express parser for [INI](https://en.wikipedia.org/wiki/INI_file) files.

This package builds with Swift Package Manager of Swift 4 Tool Chain and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project but can be used as an independent module.

## Quick Start

Configure Package.swift:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-INIParser.git", majorVersion: 3)
```

Import library into your code:

``` swift
import INIParser
```

Load the objective INI file by initializing a `INIParser` object:

``` swift
let ini = try INIParser("/path/to/somefile.ini")
```

Then it should be possible to access variables inside the file. 

### Variables with Specific Section

For most regular lines under a certain section, use `sections` attribute of `INIParser`. Take example:

```
[GroupA]
myVariable = myValue
```

Then `let v = ini.sections["[GroupA]"]?["myVariable"]` will get the value as `"myValue"`.

### Variables without Section

However, some ini files may not have any available sections but directly put all variables together:

```
freeVar1 = 1
```

In this case, call `anonymousSection` to load the corresponding value:

```
let v = ini.anonymousSection["freeVar1"]
```
