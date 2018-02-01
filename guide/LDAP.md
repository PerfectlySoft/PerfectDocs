# Perfect-LDAP


This project provides an express OpenLDAP class wrapper which enable access to OpenLDAP servers and Windows Active Directory server.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project.

Ensure you have installed and activated the latest Swift 4.0 tool chain.

*Caution*: for the reason that LDAP is widely using in many different operating systems with variable implementations, API marked with (⚠️EXPERIMENTAL⚠️) indicates that this method might not be fully applicable to certain context. However, as an open source software library, you may modify the source code to meet a specific requirement.

## Quick Start

Add the following dependency to your project's Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-LDAP.git", majorVersion: 3)
```

Then import PerfectLDAP to your source code:

``` swift
import PerfectLDAP
```

## Connect to LDAP Server

You can create actual connections as need with or without login credential. The full API is `LDAP(url:String, loginData: Login?, codePage: Iconv.CodePage)`. The `codePage` option is for those servers applying character set other than .UTF8, e.g., set `codePage: .GB2312` to connect to LDAP server in Simplified Chinese.

### TLS Option

PerfectLDAP provides TLS options for network security considerations, i.e, you can choose either `ldap://` or `ldaps://` for connections, as demo below:

``` swift
// this will connect to a 389 port without any encryption
let ld = try LDAP(url: "ldap://perfect.com")
```
or,

``` swift
// this will connect to a 636 port with certificates
let ld = try LDAP(url: "ldaps://perfect.com")
```

### Connection Timeout

Once connected, LDAP object could be set with timeout option, and the timing unit is second:

``` swift
// set the timeout for communication. In this example, connection will be timeout in ten seconds.
connection.timeout = 10
```

### Login or Anonymous

Many servers mandate login before performing any actual LDAP operations, so PerfectLDAP provides multiple login options as demo below:

``` swift
// this snippet demonstrate how to connect to LDAP server with a login credential
// NOTE: this kind of connection will block the thread until server return or timeout.
// create login credential
let credential = LDAP.login( ... )
let connection = try LDAP(url: "ldaps://...", loginData: login)
```
Aside from the above synchronous login option, a two-phased threaded login process could also bring more controls to the application:

``` swift
// first create a connection
let connection = try LDAP(url: "ldaps:// ...")

// setup login info
let credential = LDAP.login( ... )

// login in a separated thread
connection.login(info: credential) { err in
  // if err is not nil, then something must be wrong in the login process.
}
```

## Login Options

PerfectLDAP provides a special object called `LDAP.Login` to store essential account information for LDAP connections and the form of constructor is subject to the authentication types:

### Simple Login

To use simple login method, simply call `LDAP.login(binddn: String, password: String)`, as shown below:

``` swift
let credential = LDAP.Login(binddn: "CN=judy,CN=Users,DC=perfect,DC=com", password: "0penLDAP")
```

### GSSAPI

To apply GSSAPI authentication, call `LDAP.login(user:String, mechanism: AuthType)` to construct a login credential (assuming the user has already acquired a valid ticket):

``` swift
// this call will generate a GSSAPI login credential
let credential = LDAP.login(user: "judy", mechanism: .GSSAPI)
```

### GSS-SPNEGO and Digest-MD5 (⚠️EXPERIMENTAL⚠️)

To apply other SASL mechanisms, such as GSS-SPNEGO and Digest-MD5 interactive logins, call `LDAP.login(authname: String, user: String, password: String, realm: String, mechanism: AuthType)` as shown below:

``` swift
// apply DIGEST-MD5 mechanism.
let credential = LDAP.Login(authname: "judy", user: "DN:CN=judy,CN=Users,DC=perfect,DC=com", password: "0penLDAP", realm: "PERFECT.COM", mechanism: .DIGEST)
```
*⚠️NOTE⚠️* The `authname` is equivalent to `SASL_CB_AUTHNAME` and `user` is actually the macro of `SASL_CB_USER`. If any parameter above is not applicable to your case, simply assign an empty string "" to ignore it.

## Search

PerfectLDAP provides asynchronous and synchronous version of searching API with the same parameters:

### Synchronous Search

Synchronous search will block the thread until server returns, the full api is `LDAP.search(base:String, filter:String, scope:Scope, attributes: [String], sortedBy: String) throws -> [String:[String:Any]]`. Here is an example:

``` swift
// perform an ldap search synchronously, which will return a full set of attributes
// with a natural (unsorted) order, in form of a dictionary.
let res = try connection.search(base: "CN=Users,DC=perfect,DC=com", filter:"(objectclass=*)")

print(res)
```

### Asynchronous Search

Asynchronous search allows performing search in an independent thread. Once completed, the thread will call back with the result set in a dictionary. Full api of asynchronous search is `LDAP.search(base:String, filter:String, scope:Scope, attributes: [String], sortedBy: String,  completion: @escaping ([String:[String:Any]])-> Void)`. The equivalent example is:

``` swift
// perform an ldap search asynchronously, which will return a full set of attributes
// with a natural (unsorted) order, in form of a dictionary.
connection.search(base: "CN=Users,DC=perfect,DC=com", filter:"(objectclass=*)") {
  res in
  print(res)
}
```

### Parameters of Search

- base: String, search base domain (dn), default = ""
- filter: String, the filter of query, default is `"(objectclass=*)"`, means all possible results
- scope: Searching Scope, i.e., .BASE, .SINGLE_LEVEL, .SUBTREE or .CHILDREN
- sortedBy: a sorting string, may also be generated by `LDAP.sortingString()`
- completion: callback with a parameter of dictionary, empty if failed

#### Server Side Sort (⚠️EXPERIMENTAL⚠️)
The `sortedBy` parameter is a string that instructs the remote server to perform search with a sorted set. PerfectLDAP provides a more verbal way to build such a string, i.e, an array of tuples to describe what attributes would control the result set:

``` swift
// each tuple consists two parts: the sorting field and its order - .ASC or .DSC
let sort = LDAP.sortingString(sortedBy: [("description", .ASC)])
```

### Limitation of Searching Result

Once connected, LDAP object could be set with an limitation option - `LDAP.limitation`. It is an integer which specifies the maximum number of entries that can be returned on a search operation.

``` swift
// set the limitation for searching result set. In this example, only the first 1000 entries will return.
connection.limitation = 1000
```

## Attribute Operations

PerfectLDAP provides add() / modify() and delete() for attributes operations with both synchronous and asynchronous options.

### Add Attributes (⚠️EXPERIMENTAL⚠️)

Function `LDAP.add()` can add attributes to a specific DN with parameters below:

- distinguishedName: String, specific DN
- attributes:[String:[String]], attributes as an dictionary to add. In this dictionary, every attribute, as a unique key in the dictionary, could have a series of values as an array.

Both asynchronous add() and synchronous add() share the same parameters above, for example:

``` swift
// try an add() synchronously.
do {
  try connection.add(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["mail":["judy@perfect.com", "judy@perfect.org"]])
}catch (let err) {
    // failed for some reason
}

// try and add() asynchronously:
connection.add(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["mail":["judy@perfect.com", "judy@perfect.org"]]) { err in
  // if nothing wrong, err will be nil
}
```

### Modify Attributes

Function `LDAP.modify()` can modify attributes from a specific DN with parameters below:

- distinguishedName: String, specific DN
- attributes:[String:[String]], attributes as an dictionary to modify. In this dictionary, every attribute, as a unique key in the dictionary, could have a series of values as an array.

Both asynchronous modify() and synchronous modify() share the same parameters above, for example:

``` swift
// try an modify() synchronously.
do {
  try connection.modify(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["codePage":["437"]])
}catch (let err) {
    // failed for some reason
}

// try and modify() asynchronously:
connection.modify(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes:["codePage":["437"]]) { err in
  // if nothing wrong, err will be nil
}
```

### Delete Attributes (⚠️EXPERIMENTAL⚠️)

Function `LDAP.delete()` can delete attributes from a specific DN with only one parameter:

- distinguishedName: String, specific DN

Both asynchronous delete() and synchronous delete() share the same parameter above, for example:

``` swift
// try an delete() synchronously.
do {
  try connection.delete(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com")
}catch (let err) {
    // failed for some reason
}

// try and delete() asynchronously:
connection.delete(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com") { err in
  // if nothing wrong, err will be nil
}
```
