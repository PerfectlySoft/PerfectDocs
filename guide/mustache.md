# Mustache Template Support

Mustache is a logic-less templating system. It permits you to use pre-written text files with placeholders that will be replaced at run-time with values particular to a given request.

For more general information on Mustache, consult the [mustache specification](https://mustache.github.io/mustache.5.html).

To use this module, add this project as a dependency in your Package.swift file.

```swift
.package(url: "https://github.com/PerfectlySoft/Perfect-Mustache.git", from: "3.0.0")
```

Next, add "PerfectMustache" to the list of dependencies for the target that will use PerfectMustache.

Example target configuration:

```swift
targets: [.target(name: "PerfectTemplate", dependencies: ["PerfectHTTPServer", "PerfectMustache"])]
```

Then import the Mustache Module in your source code before using:

``` swift
import PerfectMustache
```

Mustache templates can be used in either an HTTP server handler or standalone with no server.

### Mustache Server Handler

To use a mustache template as an HTTP response, you will need to create a handler object which conforms to ```MustachePageHandler```. These handler objects generate the values which the template processor will use to produce its content.

```swift
/// A mustache handler, which should be passed to `mustacheRequest`, generates values to fill a mustache template
/// Call `context.extendValues(with: values)` one or more times and then
/// `context.requestCompleted(withCollector collector)` to complete the request and output the resulting content to the client.
public protocol MustachePageHandler {
	/// Called by the system when the handler needs to add values for the template.
	func extendValuesForResponse(context contxt: MustacheWebEvaluationContext, collector: MustacheEvaluationOutputCollector)
}
```

The template page handler, which you would implement, might look like the following:

```swift
struct TestHandler: MustachePageHandler { // all template handlers must inherit from PageHandler
	// This is the function which all handlers must impliment.
	// It is called by the system to allow the handler to return the set of values which will be used when populating the template.
	// - parameter context: The MustacheWebEvaluationContext which provides access to the HTTPRequest containing all the information pertaining to the request
	// - parameter collector: The MustacheEvaluationOutputCollector which can be used to adjust the template output. For example a `defaultEncodingFunc` could be installed to change how outgoing values are encoded.
	func extendValuesForResponse(context contxt: MustacheWebEvaluationContext, collector: MustacheEvaluationOutputCollector) {
		var values = MustacheEvaluationContext.MapType()
		values["value"] = "hello"
		/// etc.
		contxt.extendValues(with: values)
		do {
			try contxt.requestCompleted(withCollector: collector)
		} catch {
			let response = contxt.webResponse
			response.status = .internalServerError
			response.appendBody(string: "\(error)")
			response.completed()
		}
	}
}
```

To direct a web request to a Mustache template, call the ```mustacheRequest``` function. This function is defined as follows:

```swift
public func mustacheRequest(request req: HTTPRequest, response: HTTPResponse, handler: MustachePageHandler, templatePath: String)
```

Pass to this function the current request and response objects, your ```MustachePageHandler```, and the path to the template file you wish to serve. ```mustacheRequest``` will perform the initial steps such as creating the Mustache template parser, locating the template file, and calling your Mustache handler to generate the values which will be used when completing the template content.

The following snippet illustrates how to use a Mustache template in your URL handler. In this example, the template named "test.html" would be located in your server's web root directory:

```swift
{
	request, response in 
	let webRoot = request.documentRoot
	mustacheRequest(request: request, response: response, handler: TestHandler(), templatePath: webRoot + "/test.html")
}
```

Look at the [UploadEnumerator](https://github.com/PerfectExamples/Perfect-UploadEnumerator) example for a more concrete example.

### Standalone Usage

It is possible to use this Mustache processor in a non-web, standalone manner. You can accomplish this by either providing the path to a template file, or by supplying the template data as a string. In either case, the template content will be parsed, and any values that you supply will be filled in.

The first example uses raw template text as the source. The second example passes in a file path for the template:

```swift
let templateText = "TOP {\n{{#name}}\n{{name}}{{/name}}\n}\nBOTTOM"
let d = ["name":"The name"] as [String:Any]
let context = MustacheEvaluationContext(templateContent: templateText, map: d)
let collector = MustacheEvaluationOutputCollector()
let responseString = try context.formulateResponse(withCollector: collector)
XCTAssertEqual(responseString, "TOP {\n\nThe name\n}\nBOTTOM")
```

```swift
let templatePath = "path/to/template.mustache"
let d = ["name":"The name"] as [String:Any]
let context = MustacheEvaluationContext(templatePath: templatePath, map: d)
let collector = MustacheEvaluationOutputCollector()
let responseString = try context.formulateResponse(withCollector: collector)
```

### Tag Support

This mustache template processor supports:

* {{regularTags}}
* {{{unencodedTags}}}
* {{& unescapedTags}}
* {{# sections}} ... {{/sections}}
* {{^ invertedSections}} ... {{/invertedSections}}
* {{! comments}}
* {{> partials}}
* lambdas

**Partials**

All files used for partials must be located in the same directory as the calling template. Additionally, all partial files *must* have the file extension of **.mustache**, but this extension must not be included in the partial tag itself. For example, to include the contents of the file *foo.mustache*, you would use the tag ```{{> foo }}```.

**Encoding**

By default, all encoded tags (i.e. regular tags) are HTML-encoded, and &lt; &amp; &gt; entities will be escaped. In your handler you can manually set the ```MustacheEvaluationOutputCollector.defaultEncodingFunc``` function to perform whatever encoding you need. For example, when outputting JSON data you would want to set this function to something like the following:

```swift
collector.defaultEncodingFunc = { 
	string in 
	return (try? string.jsonEncodedString()) ?? "bad string"
}
```

**Lambdas**

Functions can be added to the values dictionary. These will be executed and the results will be added to the template output. Such functions should have the following signature:

```swift
(tag: String, context: MustacheEvaluationContext) -> String
```

The ```tag``` parameter will be the tag name. For example, the tag {{name}} would give you the value "name" for the tag parameter.
