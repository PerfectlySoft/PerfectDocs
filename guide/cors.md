## CORS (Cross Origin Resource Sharing) Security

Cross-Origin Resource Sharing (CORS) is an important part of the "open web" - and as such no framework is complete without enabling support for CORS.

Monsur Hossain from [html5rocks.com introduces CORS very effectivey](https://www.html5rocks.com/en/tutorials/cors/):

> APIs are the threads that let you stitch together a rich web experience. But this experience has a hard time translating to the browser, where the options for cross-domain requests are limited to techniques like JSON-P (which has limited use due to security concerns) or setting up a custom proxy (which can be a pain to set up and maintain).
> 
> Cross-Origin Resource Sharing (CORS) is a W3C spec that allows cross-domain communication from the browser. By building on top of the XMLHttpRequest object, CORS allows developers to work with the same idioms as same-domain requests.
> 
> The use-case for CORS is simple. Imagine the site alice.com has some data that the site bob.com wants to access. This type of request traditionally wouldn’t be allowed under the browser’s same origin policy. However, by supporting CORS requests, alice.com can add a few special response headers that allows bob.com to access the data.
> 
> As you can see from this example, CORS support requires coordination between both the server and client. Luckily, if you are a client-side developer you are shielded from most of these details. The rest of this article shows how clients can make cross-origin requests, and how servers can configure themselves to support CORS.

The [Perfect Sessions](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) module includes support for CORS configuration, to enable your API and assets to be available or secured in the way you wish.

If you have included [Perfect Sessions](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) or any of the datasource-specific implementations in your Packages.Swift file, you already have CORS support - however it is **off** by default.

### Configuration

``` swift
// Enabled, true or false.
// Default is false.
SessionConfig.CORS.enabled = true

// Array of acceptable hostnames for incoming requets
// To enable CORS on all, have a single entry, *
SessionConfig.CORS.acceptableHostnames = ["*"]

// However if you wish to enable specific domains:
SessionConfig.CORS.acceptableHostnames.append("http://www.test-cors.org")

// Wildcards can also be used at the start or end of hosts
SessionConfig.CORS.acceptableHostnames.append("*.example.com")
SessionConfig.CORS.acceptableHostnames.append("http://www.domain.*")

// Array of acceptable methods
public var methods: [HTTPMethod] = [.get, .post, .put]

// An array of custom headers allowed
public var customHeaders = [String]()

// Access-Control-Allow-Credentials true/false.
// Standard CORS requests do not send or set any cookies by default. 
// In order to include cookies as part of the request enable the client to do so by setting to true
public var withCredentials = false

// Max Age (seconds) of request / OPTION caching.
// Set to 0 for no caching (default)
public var maxAge = 3600

```

When a CORS request is submitted to the server, if there is no match then the CORS specific headers are not generated in the OPTIONS response, which will instruct the browser if cannot accept the resource.

If the server determines that CORS headers should be generated, the following headers are sent with the response:

``` swift
// An array of allowable HTTP Methods.
// In the case of the above configuration example:
Access-Control-Allow-Methods: GET, POST, PUT

// If the origin is acceptable the origin will be echoed back to the requester 
// (even if configured with *)
Access-Control-Allow-Origin: http://www.test-cors.org

// If the server wishes cookies to be sent along with requeusts, 
// this should return true
Access-Control-Allow-Credentials: true

// The length of time the OPTIONS request can be cached for.
Access-Control-Max-Age: 3600
```

An excellent resource for testing CORS entitlements and responses is available at [http://www.test-cors.org](http://www.test-cors.org)