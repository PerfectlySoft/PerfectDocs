## CSRF (Cross Site Request Forgery) Security

Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they're currently authenticated. **CSRF attacks specifically target state-changing requests, not theft of data, since the attacker has no way to see the response to the forged request.** With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker's choosing. If the victim is a normal user, a successful CSRF attack can force the user to perform state changing requests like transferring funds, changing their email address, and so forth. If the victim is an administrative account, CSRF can compromise the entire web application. [1] - (OWASP)

CSRF as an attack vector is often overlooked, and represents a significat "chaos" factor unless the validation is handled at the highest level - the framework. This will enable web application and API authors to have significant control over a vital layer of security.

The [Perfect Sessions](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) module includes support for CSRF configuration.

If you have included [Perfect Sessions](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) or any of the datasource-specific implementations in your Packages.Swift file, you already have CSRF support.

## Relevant Examples

* [Perfect-Session-Memory-Demo](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)


## Configuration

An example CSRF Configuration might look like this:

``` swift 
SessionConfig.CSRF.checkState = true
SessionConfig.CSRF.failAction = .fail
SessionConfig.CSRF.checkHeaders = true
SessionConfig.CSRF.acceptableHostnames.append("http://www.example.com")
SessionConfig.CSRF.requireToken = true
```

### SessionConfig.CSRF.checkState

This is the "master switch" - if enabled, CSRF will be enabled for all routes.

### SessionConfig.CSRF.failAction

This specifies the action to take if the CSRF validation fails. The possible options are: 

* `.fail` - Execute an immediate halt. No further processing will be done, and an HTTP Status `406 Not Acceptable` is generated.
* `.log` - Processing will continue, however the event will be recorded in the log.
* `.none` - Processing will continue, no action is taken.

### SessionConfig.CSRF.acceptableHostnames

An array of host names that are compared in the following section for "origin" match acceptance. 


### SessionConfig.CSRF.checkHeaders

If the `CORS.checkheader` is configured as `true`, origin and host headers are checked for validity.

* The `Origin`, `Referrer` or `X-Forwarded-For` headers must be populated ("origin").
* If the "origin" is spcified in `SessionConfig.CSRF.acceptableHostnames`, the CSRF check will continue to the next phase and the following checks are skipped.
* The `Host` or `X-Forwarded-Host` header must be present ("host").
* The "host" and "origin" values must match exactly.



### SessionConfig.CSRF.requireToken

When set to true, this setting will enforce all POST requests to include a "_csrf" param, or if the content type header is "application/json" then an associated "X-CSRF-Token" header must be sent with the request. The content of the header or parameter should be matching the `request.session.data["csrf"]` value. This value is set automatically at session start.

### Session state

While not a configuration param, it is worth noting that if `SessionConfig.CSRF.checkState` is true, no POST request will be accepted if the session is "new". This is a deliberate position supported by security recommendations.


 


[1] - OWASP, Cross-Site Request Forgery (CSRF): [https://www.owasp.org/index.php/Cross-Site\_Request\_Forgery\_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))