# Web Redirects

The Perfect WebRedirects module will filter for specified routes (including trailing wildcard routes) and perform redirects as instructed if a match is found.

This can be important for maintaining SEO ranking in systems that have moved. For example, if moving from a static HTML site where `/about.html` is replaced with the new route `/about` and no valid redirect is in place, the site or system will lose SEO ranking.

A demo showing the usage, and working of the Perfect WebRedirects module can be found at [Perfect-WebRedirects-Demo](https://github.com/PerfectExamples/Perfect-WebRedirects-Demo).

## Including in your project

Import the dependency into your project by specifying it in your project's Package.swift file, or adding it via Perfect Assistant.

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-WebRedirects", majorVersion: 3),
```

Then in your `main.swift` file where you configure your web server, add it as an import, and add the filter:

``` swift
import PerfectWebRedirects
```

Adding the filter:

``` swift
// Add to the "filters" section of the config:
[
	"type":"request",
	"priority":"high",
	"name":WebRedirectsFilter.filterAPIRequest,
]
```

If you are also adding Request Logger filters, if the Web Redirects object is added second, directly after the RequestLogger filter, then both the original request (and its associated redirect code) and the new request will be logged correctly.

## Configuration file

The configuration for the routes is included in JSON files at `/config/redirect-rules/*.json` in the form:

```
{

  "/test/no": {
	"code": 302,
	"destination": "/test/yes"
  },

	"/test/no301": {
		"code": 301,
		"destination": "/test/yes"
  },
  
	"/test/wild/*": {
		"code": 302,
		"destination": "/test/wildyes"
  },

	"/test/wilder/*": {
		"code": 302,
		"destination": "/test/wilding/*"
  }

}
```

Note that multiple JSON files can exist in this directory; all will be loaded the first time the filter is invoked.

The "key" is the matching route (the "old" file or route), and the "value" contains the HTTP code and new destination route to redirect to.