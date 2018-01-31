# Sessions

Perfect includes session drivers for Redis, PostgreSQL, MySQL, SQLite3, CouchDB and MongoDB servers, as well as in-memory session storage for development purposes.

Session management is a foundational function for any web or application environment, and can provide linkage to authentication, transitional preference storage, and transactional data such as for a traditional shopping cart.

The general principle is that when a user visits a web site or system with a browser, the server assigns the user a "token" or "session id" and passes this value back to the client/browser in the form of a cookie or JSON data. This token is then included as either a cookie or Bearer Token with every subsequent request.

Sessions have an expiry time, usually in the form of an "idle timeout". This means that if a session has not been active for a set number of seconds, the session is considered expired and invalid.

The Perfect Sessions implementation stores the date and time each was created, when it was last "touched", and an idle time. On each access by the client/browser the session is "touched" and the idle time is reset. If the "last touched" plus "idle time" is less than the current date/time then the session has expired.

Each session has the following properties:

* **token** - the session id
* **userid** - an optionally stored user id string
* **created** - an integer representing the date/time created, in seconds
* **updated** - an integer representing the date/time last touched, in seconds
* **idle** - an integer representing the number of seconds the session can be idle before being considered expired
* **data** - a [String:Any] Array that is converted to JSON for storage. This is intended for storage of simple preference values.
* **ipaddress** - the IP Address (v4 or v6) that the session was first used on. Used for optional session verification.
* **useragent** - the User Agent string that the session was first used with. Used for optional session verification.
* **CSRF** - the [CSRF (Cross Site Request Forgery)](csrf.md) security configuration.
* **CORS** - the [CORS (Cross Origin Resource Sharing)](cors.md) security configuration.

## Examples

Each of the modules has an associated example/demo. Much of the functionality described in this document can be observed in each of these examples.

* [In-Memory Sessions](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)
* [Redis Sessions](https://github.com/PerfectExamples/Perfect-Session-Redis-Demo)
* [PostgreSQL Sessions](https://github.com/PerfectExamples/Perfect-Session-PostgreSQL-Demo)
* [MySQL Sessions](https://github.com/PerfectExamples/Perfect-Session-MySQL-Demo)
* [SQLite Sessions](https://github.com/PerfectExamples/Perfect-Session-SQLite-Demo)
* [CouchDB Sessions](https://github.com/PerfectExamples/Perfect-Session-CouchDB-Demo)
* [MongoDB Sessions](https://github.com/PerfectExamples/Perfect-Session-MongoDB-Demo)

## Installation

If using the in-memory driver, import the base module by including the following in your project's Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session.git", majorVersion: 3)
```

### Database-Specific Drivers

Redis:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-Redis.git", majorVersion: 3)
```

PostgreSQL:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-PostgreSQL.git", majorVersion: 3)
```

MySQL:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MySQL.git", majorVersion: 3)
```

SQLite3:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-SQLite.git", majorVersion: 3)
```

CouchDB:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-CouchDB.git", majorVersion: 3)
```

MongoDB:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MongoDB.git", majorVersion: 3)
```


## Configuration

The struct `SessionConfig` contains settings that can be customized to your own preference:

``` swift
// The name of the session. 
// This will also be the name of the cookie set in a browser.
SessionConfig.name = "PerfectSession"

// The "Idle" time for the session, in seconds.
// 86400 is one day.
SessionConfig.idle = 86400

// Optional cookie domain setting
SessionConfig.cookieDomain = "localhost"

// Optional setting to lock session to the initiating IP address. Default is false
SessionConfig.IPAddressLock = true

// Optional setting to lock session to the initiating user agent string. Default is false
SessionConfig.userAgentLock = true

// The interval at which stale sessions are purged from the database
SessionConfig.purgeInterval = 3600 // in seconds. Default is 1 hour.

// CouchDB-Specific
// The CouchDB database used to store sessions
SessionConfig.couchDatabase = "sessions"

// MongoDB-Specific
// The MongoDB collection used to store sessions
SessionConfig.mongoCollection = "sessions"
```

If you wish to change the SessionConfig values, you mush set these before the Session Driver is defined.

The [**CSRF** (Cross Site Request Forgery)](csrf.md) security configuration and [**CORS** (Cross Origin Resource Sharing)](cors.md) security configuration are discussed separately.

Note that for the Redis session driver there is no "timed event" via the `SessionConfig.purgeInterval` as the mechanism for expiry is handled directly via the `Expires` value added at session creation or update.

### IP Address and User Agent Locks

If the `SessionConfig.IPAddressLock` or `SessionConfig.userAgentLock` settings are true, then the session will be forcibly ended if the incoming information does not match that which was sent when the session was initiated. 

This is a security measure to assist in preventing man-in-the-middle / session hijacking attacks.

Be aware that if a user has logged on through a WiFi network and transitions to a mobile or wired connection while the `SessionConfig.IPAddressLock` setting has been set to `true`, the user will be logged out of their session.

### Defining the Session Driver

Each Session Driver has its own implementation which is optimized for the storage option. Therefore you must set the session driver and HTTP filters before executing "server.start()".

In main.swift, after any SessionConfig changes, set the following:

``` swift 
// Instantiate the HTTPServer
let server = HTTPServer()

// Define the Session Driver
let sessionDriver = SessionMemoryDriver()

// Add the filters so the pre- and post- route actions are executed.
server.setRequestFilters([sessionDriver.requestFilter])
server.setResponseFilters([sessionDriver.responseFilter])
```

In Perfect, filters are analogous in some ways to the concept of "middleware" in other frameworks. The "request" filter will intercept the incoming request and extract the session token, attempt to load the session from storage, and make the session data available to the application. The response will be executed immediately before the response is returned to the client/browser and saves the session, and re-sends the cookie to the client/browser.

See "Database-Specific Options" below for storage-appropriate settings and Driver syntax.

### Accessing the Session Data

The `token`, `userid` and `data` properties of the session are exposed in the "request" object which is passed into all handlers. The `userid` and `data` properties can be read and written to during the scope of the handler, and automatically saved to storage in the response filter.

A sample handler can be seen in the example systems. 

``` swift
// Defining an "Index" handler
open static func indexHandlerGet(request: HTTPRequest, _ response: HTTPResponse) {
	// Random generator from TurnstileCrypto
	let rand = URandom()
	
	// Adding some random data to the session for demo purposes
	request.session.data[rand.secureToken] = rand.secureToken

	// For demo purposes, dumping all current session data as a JSON string.
	let dump = try? request.session.data.jsonEncodedString()

	// Some simple HTML that displays the session token/id, and the current data as JSON.
	let body = "<p>Your Session ID is: <code>\(request.session.token)</code></p><p>Session data: <code>\(dump)</code></p>"

	// Send the response back.
	response.setBody(string: body)
	response.completed()
}

```

To read the current session id:

``` swift
request.session.token
```

To access the userid:

``` swift
// read:
request.session.userid

// write:
request.session.userid = "MyString"
```

To access the data stored in the session:

``` swift
// read:
request.session.data

// write:
request.session.data["keyString"] = "Value"
request.session.data["keyInteger"] = 1
request.session.data["keyBool"] = true

// reading a specific value
if let val = request.session.data["keyString"] as? String {
	let keyString = val
}
```


## Database-Specific options

### Redis

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-Redis.git", majorVersion: 3)
```

Defining the connection to the PostgreSQL server:

``` swift
RedisSessionConnector.host = "localhost"
RedisSessionConnector.port = 5432
RedisSessionConnector.password = "secret"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionRedisDriver()
```

### PostgreSQL

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-PostgreSQL.git", majorVersion: 3)
```

Defining the connection to the PostgreSQL server:

``` swift
PostgresSessionConnector.host = "localhost"
PostgresSessionConnector.port = 5432
PostgresSessionConnector.username = "username"
PostgresSessionConnector.password = "secret"
PostgresSessionConnector.database = "mydatabase"
PostgresSessionConnector.table = "sessions"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionPostgresDriver()
```

### MySQL

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MySQL.git", majorVersion: 3)
```

Defining the connection to the MySQL server:

``` swift
MySQLSessionConnector.host = "localhost"
MySQLSessionConnector.port = 3306
MySQLSessionConnector.username = "username"
MySQLSessionConnector.password = "secret"
MySQLSessionConnector.database = "mydatabase"
MySQLSessionConnector.table = "sessions"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionMySQLDriver()
```

### SQLite

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-SQLite.git", majorVersion: 3)
```

Defining the connection to the SQLite server:

``` swift
SQLiteConnector.db = "./SessionDB"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionSQLiteDriver()
```

### CouchDB

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-CouchDB.git", majorVersion: 3)
```

Defining the CouchDB database to use for session storage:

``` swift
SessionConfig.couchDatabase = "perfectsessions"
```

Defining the connection to the CouchDB server:

``` swift
CouchDBConnection.host = "localhost"
CouchDBConnection.username = "username"
CouchDBConnection.password = "secret"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionCouchDBDriver()
```

### MongoDB

Importing the module, in Package.swift:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Session-MongoDB.git", majorVersion: 3)
```

Defining the MongoDB database to use for session storage:

``` swift
SessionConfig.mongoCollection = "perfectsessions"
```

Defining the connection to the MongoDB server:

``` swift
MongoDBConnection.host = "localhost"
MongoDBConnection.database = "perfect_testing"
```

Defining the Session Driver:

``` swift
let sessionDriver = SessionMongoDBDriver()
```
