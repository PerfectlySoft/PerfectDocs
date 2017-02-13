# Using Form Data

In a REST application, there are several common HTTP "verbs" that are used. The most common of these are the "GET" and "POST" verbs.

> The best-practice assignment of when to use each verb can vary between methodologies and is beyond the scope of this documentation.

An HTTP "GET" request only passes parameters in the URL:

```
http://www.example.com/page.html?message=Hello,%20World!
```

The "query parameters" in the above example are accessed using the `.queryParams` method:

``` swift
let params = request.queryParams
```

While the above example only refers to a GET request, the `.queryParams` method applies to any HTTP request as they all can contain query parameters.

## POST Parameters

POST parameters, or params, are the standard method for passing complex data between browsers and other sources to APIs for creating or modifying content.

 Perfect’s HTTP libraries make it easy to access arrays of POST params or specific params.

To return all params (Query or POST) as a `[(String,String)]` array:

``` swift
let params = request.params()
```

To return only POST params as a `[(String,String)]` array:

``` swift
let params = request.postParams()
```


To return all params with a specific name such as multiple checkboxes, type:

``` swift
let params = request.postParams(name: <String>)
```
This returns an array of strings: `[String]`

To return a specific parameter, as an optional `String?`:

``` swift
let param = request.param(name: <String>)
```

When supplying a POST parameter in the `request` object is optional, it can be useful to specify a default value if one is not supplied. In this case, use the following syntax to return an optional `String?`:

``` swift
let param = request.param(name: <String>, defaultValue: <string>)
```
