# Turnstile Authentication Layer

Turnstile is a security framework for Swift inspired by [Apache Shiro](http://shiro.apache.org/). It's used to manage the currently executing user account in your application, whether iOS app or backend web application.

The base library written by Edward Jiang from Stormpath is not specific to a framework, however additional work has been done to make things easy to integrate in a Perfect application.

Datasource-specific implementations are available, which can be forked and extended as needed. They include basic JSON and Web routes for logging a user in, registering, and checking authentication.

Turnstile "Realms" have been written for these implementations, as well as session drivers. Web session persistence is handled via cookies, and JSON session persistence is handled via header bearer tokens.

Documentation for implementing Turnstile directly is available via the [Stormpath/Turnstile](https://github.com/stormpath/Turnstile) GitHub repo.

## Turnstile-Perfect Datasource Integrations

Perfect supplies some datasource-specific integrations that can be dropped in place and used, or forked and modified to suit your project's specific needs.

* [Perfect Turnstile PostgreSQL](https://github.com/PerfectlySoft/Perfect-Turnstile-PostgreSQL) ::  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo)
* [Perfect Turnstile SQLite](https://github.com/PerfectlySoft/Perfect-Turnstile-SQLite) ::  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo)
* [Perfect Turnstile MySQL](https://github.com/PerfectlySoft/Perfect-Turnstile-MySQL) ::  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-MySQL-Demo)
* [Perfect Turnstile CouchDB](https://github.com/PerfectlySoft/Perfect-Turnstile-CouchDB) ::  [Demo](https://github.com/PerfectExamples/Perfect-Turnstile-CouchDB-Demo)

### Installation

In your Package.swift file, include the following line inside the dependency array:

``` swift
// PostgreSQL
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-PostgreSQL.git", majorVersion: 3)

// MySQL
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-MySQL.git", majorVersion: 3)

// SQLite
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-SQLite.git", majorVersion: 3)

// CouchDB
.Package(url: "https://github.com/PerfectlySoft/Perfect-Turnstile-CouchDB.git", majorVersion: 3)
```

### Included JSON Routes

The framework includes certain basic routes:

```
POST /api/v1/login (with username & password form elements)
POST /api/v1/register (with username & password form elements)
GET /api/v1/logout
```

### Included Routes for Browser

The following routes are available for browser testing:

```
http://localhost:8181
http://localhost:8181/login
http://localhost:8181/register
```

These routes are using Mustache files in the webroot directory.

Example Mustache files can be found in [https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-SQLite-Demo) or [https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo](https://github.com/PerfectExamples/Perfect-Turnstile-PostgreSQL-Demo).

### Creating an HTTP Server with Authentication

Setting up a complete HTTP web server with the Turnstile authentication layer and database access is easy. The following script is a complete "main.swift" starting point for your executable. A detailed breakdown of this script is included below.

``` swift 
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

import StORM
import PerfectRequestLogger
import TurnstilePerfect

// see next section for alternate daabase options
import PostgresStORM
import PerfectTurnstilePostgreSQL


// uncomment to turn on SQL Logging to console
//StORMdebug = true

RequestLogFile.location = "./requests.log"

// Used later in script for the Realm and how the user authenticates.
let pturnstile = TurnstilePerfectRealm()

// Set the connection vatiable
// Replace with your values
PostgresConnector.host        = "localhost"
PostgresConnector.username    = "perfect"
PostgresConnector.password    = "perfect"
PostgresConnector.database    = "perfect_testing"
PostgresConnector.port        = 5432

// Set up the Authentication table
let auth = AuthAccount()
try? auth.setup()

// Connect the AccessTokenStore
tokenStore = AccessTokenStore()
try? tokenStore?.setup()

// Create HTTP server.
let server = HTTPServer()

// Register routes and handlers
let authWebRoutes = makeWebAuthRoutes()
let authJSONRoutes = makeJSONAuthRoutes("/api/v1")

// Add the routes to the server.
server.addRoutes(authWebRoutes)
server.addRoutes(authJSONRoutes)

// Add more routes here
var routes = Routes()
// routes.add(method: .get, uri: "/api/v1/test", handler: AuthHandlersJSON.testHandler)

// Add the routes to the server.
server.addRoutes(routes)

// add routes to be checked for auth
var authenticationConfig = AuthenticationConfig()
authenticationConfig.include("/api/v1/*")
authenticationConfig.exclude("/api/v1/login")
authenticationConfig.exclude("/api/v1/register")

let authFilter = AuthFilter(authenticationConfig)

// Note that order matters when the filters are of the same priority level
server.setRequestFilters([pturnstile.requestFilter])
server.setResponseFilters([pturnstile.responseFilter])

server.setRequestFilters([(authFilter, .high)])

// Request and response filters for HTTP logging
server.setRequestFilters([(myLogger, .high)])
server.setResponseFilters([(myLogger, .low)])

// Set a listen port of 8181
server.serverPort = 8181

// Where to serve static files from
server.documentRoot = "./webroot"

do {
	// Launch the HTTP server.
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("Network error thrown: \(err) \(msg)")
}

```

#### Requirements

Define the "Realm" which is the Turnstile definition of how the authentication is handled. The implementation is specific to the SQLite datasource, although it is very similar between datasources and is designed to be generic and extendable.

``` swift 
let pturnstile = TurnstilePerfectRealm()
```

Define the connection parameters. If using PostgreSQL:

``` swift
// Replace with your values
PostgresConnector.host        = "localhost"
PostgresConnector.username    = "perfect"
PostgresConnector.password    = "perfect"
PostgresConnector.database    = "perfect_testing"
PostgresConnector.port        = 5432
```

Or, define the location and name of the SQLite3 database:

``` swift
connect = SQLiteConnect("./authdb")
```

Define, and initialize up the authentication table:

``` swift 
let auth = AuthAccount()
try? auth.setup()
```

Connect the AccessTokenStore:

``` swift
tokenStore = AccessTokenStore()
try? tokenStore?.setup()
```

Create the HTTP Server:

``` swift
let server = HTTPServer()
```

Register routes and handlers and add the routes to the server:

``` swift 
let authWebRoutes = makeWebAuthRoutes()
let authJSONRoutes = makeJSONAuthRoutes("/api/v1")

server.addRoutes(authWebRoutes)
server.addRoutes(authJSONRoutes)
```

Add routes to be checked for authentication:

``` swift
var authenticationConfig = AuthenticationConfig()
authenticationConfig.include("/api/v1/*")
authenticationConfig.exclude("/api/v1/login")
authenticationConfig.exclude("/api/v1/register")

let authFilter = AuthFilter(authenticationConfig)
```

These routes can be either separate, or as an array of strings. They describe inclusions and exclusions. Wildcards are supported where the wildcard character (*) is the last character in the string.

Add request & response filters. Note the order which you specify filters that are of the same priority level:

``` swift
server.setRequestFilters([pturnstile.requestFilter])
server.setResponseFilters([pturnstile.responseFilter])

server.setRequestFilters([(authFilter, .high)])

// Request and response filters for HTTP logging
server.setRequestFilters([(myLogger, .high)])
server.setResponseFilters([(myLogger, .low)])
```

Now, set the port and static files location, and start the server:

``` swift
// Set a listen port of 8181
server.serverPort = 8181

// Where to serve static files from
server.documentRoot = "./webroot"

do {
	// Launch the HTTP server.
	try server.start()
} catch PerfectError.networkError(let err, let msg) {
	print("Network error thrown: \(err) \(msg)")
}
```

## Database-specific import and configuration

### PostgreSQL

Import:

``` swift
import PostgresStORM
import PerfectTurnstilePostgreSQL
```

Configuration:

``` swift
PostgresConnector.host        = "localhost"
PostgresConnector.username    = "username"
PostgresConnector.password    = "secret"
PostgresConnector.database    = "dbname"
PostgresConnector.port        = 5432
```

### MySQL

Import:

``` swift
import MySQLStORM
import PerfectTurnstileMySQL
```

Configuration:

``` swift
MySQLConnector.host        = "localhost"
MySQLConnector.username    = "username"
MySQLConnector.password    = "secret"
MySQLConnector.database    = "dbname"
MySQLConnector.port        = 3306
```

### SQLite

Import:

``` swift
import SQLiteStORM
import PerfectTurnstileSQLite
```

Configuration:

``` swift
SQLiteConnector.db = "./mydb"
```

### CouchDB

Import:

``` swift
import CouchDBStORM
import PerfectTurnstileCouchDB
```

Configuration:

``` swift
CouchDBConnection.host = "localhost"
CouchDBConnection.username = "username"
CouchDBConnection.password = "secret"
CouchDBConnection.port = 5984
CouchDBConnection.ssl = false
```
