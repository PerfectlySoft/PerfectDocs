# Perfect Local Authentication

This package provides Local Authentication libraries for projects that require locally stored and handled authentication.

## Relevant Templates & Examples

A template application can be found at [https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template](https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template), providing a fully functional starting point, as well as demonstrating the usage of the system.

## Adding to your project

Add this project as a dependency in your Package.swift file.

PostgreSQL Driver:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-LocalAuthentication-PostgreSQL.git", majorVersion: 3)
```

MySQL Driver:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-LocalAuthentication-MySQL.git", majorVersion: 3)
```

## Configuration

It is important to configure the following in main.swift to set up database and session configuration:

Import the required modules:

``` swift
import PerfectSession
import PerfectSessionPostgreSQL // Or PerfectSessionMySQL
import PerfectCrypto
import LocalAuthentication
```

Initialize PerfectCrypto:

``` swift
let _ = PerfectCrypto.isInitialized
```

Now set some defaults:

``` swift
// Used in email communications
// The Base link to your system, such as http://www.example.com/
var baseURL = ""

// Configuration of Session
SessionConfig.name = "perfectSession" // <-- change
SessionConfig.idle = 86400
SessionConfig.cookieDomain = "localhost" //<-- change
SessionConfig.IPAddressLock = false
SessionConfig.userAgentLock = false
SessionConfig.CSRF.checkState = true
SessionConfig.CORS.enabled = true
SessionConfig.cookieSameSite = .lax
```

Detailed Session configuration documentation can be found at [https://www.perfect.org/docs/sessions.html](https://www.perfect.org/docs/sessions.html)

The database and email configurations should be set as follows (if using JSON file config):

``` swift
let opts = initializeSchema("./config/ApplicationConfiguration.json") // <-- loads base config like db and email configuration
httpPort = opts["httpPort"] as? Int ?? httpPort
baseURL = opts["baseURL"] as? String ?? baseURL
```

Otherwise, these will need to be set equivalent to one of these functions:

* PostgreSQL: [https://github.com/PerfectlySoft/Perfect-LocalAuthentication-PostgreSQL/blob/master/Sources/LocalAuthentication/Schema/InitializeSchema.swift](https://github.com/PerfectlySoft/Perfect-LocalAuthentication-PostgreSQL/blob/master/Sources/LocalAuthentication/Schema/InitializeSchema.swift)

* MySQL: [https://github.com/PerfectlySoft/Perfect-LocalAuthentication-MySQL/blob/master/Sources/LocalAuthentication/Schema/InitializeSchema.swift](https://github.com/PerfectlySoft/Perfect-LocalAuthentication-MySQL/blob/master/Sources/LocalAuthentication/Schema/InitializeSchema.swift)

### Set the session driver

PostgreSQL: 

``` swift
let sessionDriver = SessionPostgresDriver()
```

MySQL: 

``` swift
let sessionDriver = SessionMySQLDriver()
```

### Request & Response Filters

The following two session filters need to be added to your server config:

PostgreSQL Driver:

``` swift
// (where filter is a [[String: Any]] object)
filters.append(["type":"request","priority":"high","name":SessionPostgresFilter.filterAPIRequest])
filters.append(["type":"response","priority":"high","name":SessionPostgresFilter.filterAPIResponse])
```

MySQL Driver:

``` swift
// (where filter is a [[String: Any]] object)
filters.append(["type":"request","priority":"high","name":SessionMySQLFilter.filterAPIRequest])
filters.append(["type":"response","priority":"high","name":SessionMySQLFilter.filterAPIResponse])
```

For example, see [https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template/blob/master/Sources/PerfectLocalAuthPostgreSQLTemplate/configuration/Filters.swift](https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template/blob/master/Sources/PerfectLocalAuthPostgreSQLTemplate/configuration/Filters.swift)

### Add routes for login, register etc

The following routes can be added as needed or customized to add login, logout, register:

``` swift
// Login
routes.append(["method":"get", "uri":"/login", "handler":Handlers.login]) // simply a serving of the login GET
routes.append(["method":"post", "uri":"/login", "handler":LocalAuthWebHandlers.login])
routes.append(["method":"get", "uri":"/logout", "handler":LocalAuthWebHandlers.logout])

// Register
routes.append(["method":"get", "uri":"/register", "handler":LocalAuthWebHandlers.register])
routes.append(["method":"post", "uri":"/register", "handler":LocalAuthWebHandlers.registerPost])
routes.append(["method":"get", "uri":"/verifyAccount/{passvalidation}", "handler":LocalAuthWebHandlers.registerVerify])
routes.append(["method":"post", "uri":"/registrationCompletion", "handler":LocalAuthWebHandlers.registerCompletion])

// JSON
routes.append(["method":"get", "uri":"/api/v1/session", "handler":LocalAuthJSONHandlers.session])
routes.append(["method":"get", "uri":"/api/v1/logout", "handler":LocalAuthJSONHandlers.logout])
routes.append(["method":"post", "uri":"/api/v1/register", "handler":LocalAuthJSONHandlers.register])
routes.append(["method":"login", "uri":"/api/v1/login", "handler":LocalAuthJSONHandlers.login])
```

An example can be found at [https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template/blob/master/Sources/PerfectLocalAuthPostgreSQLTemplate/configuration/Routes.swift](https://github.com/PerfectlySoft/Perfect-Local-Auth-PostgreSQL-Template/blob/master/Sources/PerfectLocalAuthPostgreSQLTemplate/configuration/Routes.swift)

## Testing for authentication:

The user id can be accessed as follows:

``` swift
request.session?.userid ?? ""
```

If a user id (i.e. logged in state) is required to access a page, code such as this could be used to detect and redirect:

``` swift
let contextAuthenticated = !(request.session?.userid ?? "").isEmpty
if !contextAuthenticated { response.redirect(path: "/login") }
```