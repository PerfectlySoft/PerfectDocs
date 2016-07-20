# MySQL

The MySQL connector provides a wrapper around MySQL, allowing interaction between your Perfect Applications &amp; MySQL databases. 

## System Requirements

### macOS

Requires the use of Homebrew’s MySQL. 

```shell
brew install mysql
```

If you need home-brew, you can install it with: 

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Unfortunately, at this point in time you will need to edit the mysqlclient.pc file located here:

```shell
/usr/local/lib/pkgconfig/mysqlclient.pc
```

Remove the occurrance of "-fno-omit-frame-pointer". This file is read-only by default so you will need to change that first.

### Linux 

Ensure that you have installed libmysqlclient-dev for MySQL version 5.6 or greater:

```shell
sudo apt-get install libmysqlclient-dev
```

Please note that Ubuntu 14 defaults to including a version of MySQL client which will not compile with this package. Install MySQL client version 5.6 or greater manually. 

## Setup

Add the "Perfect-MySQL" project as a dependency in your Package.swift file:

```swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-MySQL.git", versions: Version(0,0,0)..<Version(10,0,0))
```

### Import

Fist and foremost, in any of the source files you intend to use with MySQL, import the module with: 

```swift
import mysql
```

## Quick Start