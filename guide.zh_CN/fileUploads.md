# 文件上传

这里有一个[数据上传](formData.md)的例子，用于说明如何操作文件上载。

HTTP表单数据主要采用以下两种编码格式：

* application/x-www-form-urlencoded （默认编码格式）
* multipart/form-data

如果要使用文件上传空间，则必须选择`multipart/form-data`作为表单的`enctype`编码类型。

完整的例子请查看[Perfect文件上传例子](https://github.com/iamjono/perfect-file-uploads)

典型的HTML文件上传表单中编码类型声明应该像以下例子这样：

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

## 在服务器端接收文件

因为表单是POST方法，我们需要用`.post`方法管理请求响应路由

``` swift
var routes = Routes()
routes.add(
    method: .post,
    uri: "/upload",
    handler: handler)
server.addRoutes(routes)
```

一旦对方请求内容完成文件传输，则我们可以使用请求句柄`handler`：

``` swift
// 通过操作fileUploads数组来掌握文件上传的情况
// 如果这个POST请求不是分段multi-part类型，则该数组内容为空

if let uploads = request.postFileUploads where uploads.count > 0 {
    // 创建一个字典数组用于检查已经上载的内容
    var ary = [[String:Any]]()

    for upload in uploads {
        ary.append([
            "fieldName": upload.fieldName,	//字段名
            "contentType": upload.contentType, //文件内容类型
            "fileName": upload.fileName,	//文件名
            "fileSize": upload.fileSize,	//文件尺寸
            "tmpFileName": upload.tmpFileName	//上载后的临时文件名
            ])
    }
    values["files"] = ary
    values["count"] = ary.count
}
```

如上所述，被上传的文件（一个或多个文件）可以用`request.postFileUploads`数组表示，每个数组元素都有不同的属性，如`fileName`文件名］、`fileSize`文件尺寸、`tmpFileName`临时文件名等等。

**⚠️注意⚠️** ：文件上传后会被自动放置到一个临时目录。您需要自己将临时文件转移至期望位置。

因此我们可以创建一个目录来放置这些上传来的文件。该目录因安全考虑不会和webroot的根目录放在一起：

``` swift
// 创建路径用于存储已上传文件
let fileDir = Dir(Dir.workingDir.path + "files")
do {
    try fileDir.create()
} catch {
    print(error)
}
```

下一步，在`for upload in uploads`代码段，我们会执行文件转移的操作：

``` swift
// 将文件转移走，如果目标位置已经有同名文件则进行覆盖操作。
let thisFile = File(upload.tmpFileName)
do {
    let _ = try thisFile.moveTo(path: fileDir.path + upload.fileName, overWrite: true)
} catch {
    print(error)
}
```

现在上传完毕的文件就可以按照原来的文件名转移到目标目录。

关于文件系统操作的详细内容，请参考[文件操作](dir.md) 和[目录操作](file.md)章节。
