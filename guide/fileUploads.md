# File Uploads

A special case of [using form data](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/formData.md) is handling file uploads.

There are two main form encoding types:

* application/x-www-form-urlencoded (the default)
* multipart/form-data

When you wish to include file upload elements, you must choose multipart/form-data as your form's `enctype` (encoding) type.

All code used below can be seen in action as a complete example at [https://github.com/iamjono/perfect-file-uploads](https://github.com/iamjono/perfect-file-uploads)

An example HTML form containing the correct encoding and file input element might be represented like this:

```
<form 
	method="POST" 
	enctype="multipart/form-data" 
	action="/upload">
	<input type="file" name="filetoupload">
	<br>
	<input type="submit">
</form>
```

## Receiving the File on the Server Side

Because the form is a POST method, we will handle the route with a `method: .post`:

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

As demonstrated above, the file(s) uploaded are represented by the `request.postFileUploads` array, and the various properties such as `fileName`, `fileSize` and `tmpFileName` can be accessed from each array component.

**Note: The files uploaded are placed in a temporary directory. It is your responsibility to move them into the desired location.**

So let's create a directory to hold the uploaded files. This directory is outside of the webroot directory for security reasons:

``` swift 
// create uploads dir to store files
let fileDir = Dir(Dir.workingDir.path + "files")
do {
	try fileDir.create()
} catch {
	print(error)
}
```

Next, inside the `for upload in uploads` code block, we will create the action for the file to be moved:

``` swift
// move file
let thisFile = File(upload.tmpFileName)
do {
	let _ = try thisFile.moveTo(path: fileDir.path + upload.fileName, overWrite: true)
} catch {
	print(error)
}
```

Now the uploaded files will move to the specified directory with the original filename restored.

For more information on file system manipulation, please see the [Directory Operations](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/dir.md) and [File Operations](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/file.md) chapters. 