# Perfect-Mustache
Mustache template support for Perfect.

Mustache is a logic-less templating system. It permits you to use pre-written text files with placeholders. The placeholders will be filled in a run-time with values particular to a given request.

For more general information on Mustache, consult the [mustache documentation](https://mustache.github.io/mustache.5.html)

To start, add this project as a dependency in your Package.swift file.

```swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Mustache.git", versions: Version(0,0,0)..<Version(10,0,0))
```

The following snippet illustrates how to use mustache templates in your URL handler. In this example, the template named "test.html" would be located in your server's web root directory.

```swift
{
	request, response in 
	let webRoot = request.documentRoot
	mustacheRequest(request: request, response: response, handler: TestHandler(), templatePath: webRoot + "/test.html")
}
```

The template page handler, which you would impliment, might look like the following.

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

Look at the [UploadEnumerator](https://github.com/PerfectlySoft/PerfectExample-UploadEnumerator) example for a more concrete example.

**Tag Support**

This mustache template processor supports:

* {{regularTags}}
* {{& unencodedTags}}
* {{# sections}} ... {{/sections}}
* {{^ invertedSections}} ... {{/invertedSections}}
* {{! comments}}
* {{> partials}}
* lambdas

**Partials**

All files used for partials must be located in the same directory as the calling template. Additionally, all partial files *must* have the file extension of **mustache** but this extension must not be included in the partial tag itself. For example, to include the contents of the file *foo.mustache* you would use the tag ```{{> foo }}```.

**Encoding**

By default, all encoded tags (i.e. regular tags) are HTML encoded and &lt; &amp; &gt; entities will be escaped. In your handler you can manually set the ```MustacheEvaluationOutputCollector.defaultEncodingFunc``` function to perform whatever encoding you need. For example when outputting JSON data you would want to set this function to something like the following:

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

The ```tag``` parameter will be the tag name. For example the tag {{name}} would give you the value "name" for the tag parameter.
