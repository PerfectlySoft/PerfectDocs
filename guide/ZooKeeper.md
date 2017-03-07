# Perfect-ZooKeeper

This project implements an express Swift library of ZooKeeper

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project.


## Release Note

This project can only be built on ⚠️ Ubuntu 16.04 ⚠️ . Mac OS X doesn't support it.

## Quick Start

### Swift Package Manager

Add Perfect-ZooKeeper to your project's Package.swift:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-ZooKeeper.git", majorVersion: 1)
```

### Import Library

Import Perfect-ZooKeeper library to your source code:

``` swift
import PerfectZooKeeper
```

### Debug

To debug your ZooKeeper application, Perfect-ZooKeeper provides a static method called `debug` to set the debug level, for example:

``` swift
// this will set debug level to the whole application
ZooKeeper.debug()
```

You can also adjust the debug level to method `debug(_ level: LogLevel = .DEBUG)` by a parameter:

- level: LogLevel, the debug level, could be .ERROR, .WARN, .INFO, or .DEBUG by default

### Log

To trace and log your ZooKeeper application, Perfect-ZooKeeper provides a static method called `log`, for example:

``` swift

// this will redirect the debug information to standard error stream
ZooKeeper.log()

```

The only parameter of `log(_ to: UnsafeMutablePointer<FILE> = stderr)` is `to`, a `FILE` pointer as in C stream, it is `stderr` by default but you can redirect it to any available `FILE` streams.

### Using ZooKeeper Object

Before performing any actual connections, it is necessary to construct a `ZooKeeper` object:

``` swift
let z = ZooKeeper()
```

Or alternatively, you can also add a timeout setting to such an object, which defines the maximal milliseconds to wait for connection:

``` swift
// indicates that the connection attempt will be treated as broken in eight seconds.
let z = ZooKeeper(8192)
```

### Connect to ZooKeeper Hosts

Use `ZooKeeper.connect()` to connect to specified hosts. Take example, the demo below shows how to connect to a ZooKeeper host, and how the program invokes your callback once connected:

``` swift
try z.connect("servername:2181") { connect in
  switch(connect) {
  case .CONNECTED:
    // connection is made
  case .EXPIRED:
    // connection is expired
  default:
    // connection is broken
  }
}
```

⚠️ NOTE ⚠️ , you may also connect to a cluster of host by replacing the above connection string into a string of multiple hosts in such an expression: `"server1:2181,server2:2181,server3:2181"`, but this may be subject to the ZooKeeper version that you are connecting with. For more information of ZooKeeper connection string, see [ZooKeeper Programmer's Guide](https://zookeeper.apache.org/doc/trunk/zookeeperProgrammers.html)


### Existence of a ZNode

Once connected, you may check a specific ZNode by calling `exists()`, for example:

``` swift
let a = try z.exists("/path/to")
print(a)
```

This function will return a `Stat()` structure if nothing wrong, for example:
``` swift
// this is a sample result of calling print(try z.exists("/path/to"))
Stat(czxid: 0, mzxid: 0, ctime: 0, mtime: 0, version: 0, cversion: -1, aversion: 0, ephemeralOwner: 0, dataLength: 0, numChildren: 1, pzxid: 0)
```

### List Children of a ZNode

Method `children()` may list all available direct sub nodes under the objective and put them into an array of string.

``` swift
let kids = try z.children("/path/to")
// if success, it will list all sub nodes under /path/to in an array.
// for example, if there is /path/to/a and /path/to/b,
// then the result is probably ["a", "b"]
print(kids)
```

### Save Data to a ZNode

As a key-value directory, each ZNode may contain a small amount of data in form of a string, usually not exceed to 10k. You can save your own configuration data into a ZNode, synchronously or asynchronously, as demanded.

#### Save Data Synchronously

Synchronous version of `save()` will return a `Stat()` structure if success:

``` swift
let stat = try z.save("/path/to/key", data: "my configuration value of key")
print(stat)
```

Parameters of `func save(_ path: String, data: String, version: Int = -1) throws -> Stat`:
- path: String, the absolute full path of the node to access
- data: String, the data to save
- version: Int, version of data, default is -1 which indicates ignoring the version info

#### Save Data Asynchronously

Asynchronous version of `save()` has all the same parameters with an extra `StatusCallback` but without returning value:

``` swift
try z.save("/path/to/key", data: "my configuration value of key") { err, stat in
  guard err == .ZOK else {
    // something wrong
  }
  guard let st = stat else {
    // async save() returns a null status
  }
  // print the status after saving
  print(st)
}
```

### Load Data from a ZNode

Similar to `save()`, the ZooKeeper `load()` also has both synchronous version and asynchronous version as well:

#### Load Data Synchronously

To load data from a ZNode synchronously, simply call `load("/path/to")`, and it will return a tuple of `(value: String, stat: Stat)`, which stands for data value and status of the node:

``` swift
let (value, stat) = try z.load("/path/to")
```

#### Load Data Asynchronously

The data loading from a ZNode asynchronously will require an extra callback with a parameter of (error: Exception, value: String, Stat) as demo below:

``` swift
try z.load(path) { err, value, stat in
  guard err == .ZOK else {
    // something wrong
  }//end guard
  guard let st = stat else {
    // there is no status information of node
  }//end guard
  print(st)
  // this is the actual data value as a String
  print(value)
}//end load
```

### Make a Node

Function `func make(_ path: String, value: String = "", type: NodeType = .PERSISTENT, acl: ACLTemplate = .OPEN) throws -> String` can build different type of nodes, with writing data value and set ACL (Access Control List) info for this node in the same moment. Here are the parameters:

- path: String, the absolute full path of the node to make
- value: String, the value to store into node
- type: NodeType, i.e., .PERSISTENT, .EPHEMERAL, .SEQUENTIAL, or .LEADERSHIP, which means ephemeral + sequential. Default type is .PERSISTENT
- acl: ACLTemplate, basic ACL template to apply in this incoming node, i.e., .OPEN, .READ or .CREATOR. Default is .OPEN, which means nothing to restrict

⚠️ Note ⚠️ The return value will be the newly created pat, i.e., the same one as input if type is .PERSISTENT or .EPHEMERAL, and will be appended with a serial number only if the node type is .SEQUENTIAL or .LEADERSHIP.

#### Make a Persistent Node

The following code demonstrates how to create a persistent node with data:

``` swift
let _ = try z.make("/path/to/key", value: "my config data value for this key")
```

#### Make a Temporary Node

A temporary ZNode means it will automatically disappear once the session was over (usually after a few seconds of disconnection). To create such a node, simply add a node type parameter to call:

``` swift
let _ = try z.make("/path/to/tempKey", value: "data for this temporary key", type: .EPHEMERAL)
```

#### Make a Sequential Node

A Sequential ZNode means if you want to create a `/path/to/key` node, it will return a `/path/to/key0123456789`, i.e., a 10 digit number will be added to the node you named.

``` swift
let path = try z.make("/path/to/myApplication", type: .SEQUENTIAL)
print(path)
// if success, the path will be something like `/path/to/myApplication0000000123`
```

⚠️ Note ⚠️ Sequential node is persistent and can be removed only by calling `remove()` method explicitly.

#### Make a Leadership Node

The purpose of leadership node is to select a leadership server among all candidates. Similar to .SEQUENTIAL, the leadership node is also a temporary node, which help all clustered backups to determine who shall be the leader among the cluster by checking whose serial number is the minimal one, which means who is the first available.

``` swift
let path = try z.make("/path/to/myApplication", type: .LEADERSHIP)
print(path)
// if success, the path will be something like `/path/to/myApplication0000000123`
// and will be deleted automatically once disconnected.
```

### Remove a Node

Method `func remove(_ path: String, version: Int32 = -1)` enables the function to delete an existing node:

``` swift
// this action will result in a removal regardless node versions.
try z.remove("/path/to/uselessNode")
```

### Watch for Changes

Perfect-ZooKeeper provides a useful function `watch()` to monitor other instance operations against a specific node.
The full API of `watch()` method is `func watch(_ path: String, eventType: EventType = .BOTH, renew: Bool = true, onChange: @escaping WatchCallback)` with parameter explained below:

- path: String, the absolute full path of the node to watch
- eventType: watch for .DATA or .CHILDREN, or .BOTH
- renew: watch the event for once or for ever, false for once and true for ever.
- onChange: WatchCallback, callback once something changed

For example:

``` swift
try z.watch("/path/to/myCheese") { event in
  switch(event) {
  case CONNECTED:
    // me myself just connected to this node???
  case DISCONNECTED:
    // connection is broken
  case EXPIRED:
    // connection is expired
  case CREATED:
    // this shall never happen - just created for the node me watch?
  case DELETED:
    // the node has been deleted by someone else
  case DATA_CHANGED:
    // someone just touched my cheese
  case CHILD_CHANGED:
    // children were changed
  default:
    // unexpected here
  }
}//end watch
```

### Election in a Cluster

Perfect ZooKeeper provides a convenient way of choosing a leader / master from a cluster by calling method `elect()`:

``` swift
let (me, leader, candidates) = try z.elect("/path/to")
```

If success, the return value of `elect()` function is a tuple of `(me: Int, leader: Int, candidates: [Int])`, which means every candidate, include the current instance, will be included in the `candidates` array as an integer. Result `me` is the current instance's election serial number and the result of `leader` is the number represents the final leader in this election. If `me == leader`, then congratulations - the current instance of ZooKeeper just won the election. In such a case, further actions should be done to upgrade current instance into the master of cluster.

### ACL Operations

ACL, the access control list, can be manipulated in ZooKeeper API by core data structure `ACL_vector`, i.e., a C based pointer array. Content of such a structure could be checked by similar operations as below:

``` swift
func show(_ aclArray: ACL_vector) {
  guard let pAcl = aclArray.data else {
    // the array doesn't contain any valid data
    return
  }
  var i = 0
  while (Int32(i) < aclArray.count) {
    let cursor = pAcl.advanced(by: i)
    let acl = cursor.pointee
    let scheme = String(cString: acl.id.scheme)
    let id = String(cString: acl.id.id)
    // id is subject to scheme
    // if scheme is "world", then id shall be "anyone",
    // scheme "auth" doesn't use any id.
    // scheme "ip" uses host ip as id, such as "1.2.3.4/5"
    // scheme "x509" uses client X500 principal as id.
    // scheme "digest" use name:password to generate MD5 as id:
    // `username:base64 encoded SHA1 password digest.`
    print("id: \(id)")
    print("scheme: \(scheme)")
    // permission is a combination of :
    // ZOO_PERM_READ | ZOO_PERM_WRITE | ZOO_PERM_CREATE
    // ZOO_PERM_DELETE | ZOO_PERM_ADMIN | ZOO_PERM_ALL
    let perm = String(format: "%8X", acl.perms)
    print("permissions: \(perm)")
    i += 1
  }
}
```

⚠️ CAUTION ⚠️ To manipulate ACL_vector with permission options, please make sure to import the C ZooKeeper library:

``` swift
import czookeeper
```

#### Get ACL Info

Method `getACL()` can retrieve ACL info from a ZNode:

``` swift
let (acl, stat) = try z.getACL("/path/to")
```

The return value is a tuple of `(acl: ACL_vector, stat: Stat)` stands for the acl info as an `ACL_vector` pointer array and status of the node.

#### Set ACL Info

Method `func setACL(_ path: String, version: Int32 = -1, acl: ACL_vector) throws` may save an ACL setting to a ZNode, where the version could be skipped by default and providing a valid `ACL_vector` pointer array:

``` swift
var acl = ACL_vector()
// do some modification to acl variable
try z.setACL("/path/to", acl:acl)
```

However, there is also another alternative form of `setACL()` by replacing the complicated `ACL_vector` to preset ACL template:

``` swift
// available templates include .OPEN, .READ and .CREATOR
// default is .OPEN, which means nothing to restrict
try z.setACL("/path/to", aclTemplate: .READ)
```
