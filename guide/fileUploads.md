# File Uploads

A special case of working with form data is handling file uploads.

There are two main types of form encoding types:

* application/x-www-form-urlencoded (the default)
* multipart/form-data

When you wish to include file upload elements, you must choose multipart/form-data as your form's `enctype` (encoding) type.

An example HTML form containing the correct encoding and file input element might be represented like this:

``` html
<form 
	method="POST" 
	enctype="multipart/form-data" 
	action="/upload">
	<input type="file" name="filetoupload">
	<br>
	<input type="submit">
</form>
```

## Receiving the file server-side

Because the form is a POST method, we will be handling the route with a `method: .post`:

``` swift
var routes = Routes()
routes.add(
	method: .post, 
	uri: "/upload", 
	handler: handler)
server.addRoutes(routes)
```

Once the request has been offloaded to the `handler` we can:

``` swift
// Grab the fileUploads array and see what's there
// If this POST was not multi-part, then this array will be empty

if let uploads = request.postFileUploads where uploads.count > 0 {
	// Create an array of dictionaries which will show what was uploaded
	var ary = [[String:Any]]()

	for upload in uploads {
		ary.append([
			"fieldName": upload.fieldName,
			"contentType": upload.contentType,
			"fileName": upload.fileName,
			"fileSize": upload.fileSize,
			"tmpFileName": upload.tmpFileName
			])
	}
	values["files"] = ary
	values["count"] = ary.count
}
```

As demonstrated in the code above, the file(s) uploaded are represented by the `request.postFileUploads` array, and the various properties such as `fileName`, `fileSize` and `tmpFileName` can be accessed from each array component.

**Note:** The files uploaded are placed in a temporary directory. It is your responsibility to move them into the desired location.

First, lets create the directory to hold the uploaded files. This directory is outside of the webroot directory for security:

``` swift 
// create uploads dir to store files
let fileDir = Dir(Dir.workingDir.path + "files")
do {
	try fileDir.create()
} catch {
	print(error)
}
```

Next, inside the `for upload in uploads` code block we will action the file move:

``` swift
// move file to webroot
let thisFile = File(upload.tmpFileName)
do {
	let _ = try thisFile.moveTo(path: fileDir.path + upload.fileName, overWrite: true)
} catch {
	print(error)
}
```

The result of this will be that the uploaded files will be moved to the specified directory, with the original filename restored.

For more information on file system manipulation, please see the "Dir" and "File" chapters. 