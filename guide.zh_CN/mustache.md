# Mustache页面模板

Mustache是一个弱逻辑的模板系统。该模板系统允许在预先备好的文件中用文字占位符代替目标变量值，而当真正网络请求发生的时候才会把占位符用真正的变量值替换掉。

Mustache详细说明请见[Mustache标准（英文）](https://mustache.github.io/mustache.5.html)。

为了使用这个模块，请在您的Package.swift文件中增加依存关系：

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-Mustache.git",
	majorVersion: 3
	)
```

使用之前，请在源代码开头部分增加对Mustache的引用导入：

``` swift
import PerfectMustache
```

Mustache模板可以在HTTP服务器中调用，也可以单独使用，不需要服务器。

### 服务器内调用Mustache

如果希望Mustache模板在HTTP响应中调用，您需要采用```MustachePageHandler```网页模板句柄创建一个新的实例对象。这些句柄对象将生成用于模板处理器创建内容的参数值。

``` swift
/// Mustache句柄，应该作为参数传递给`mustacheRequest`请求对象，为完成Mustache模板创建参数值
/// 将会一次或多次调用 `context.extendValues(with: values)`
/// `context.requestCompleted(withCollector collector)`用于完成请求并将结果输出给客户端。
public protocol MustachePageHandler {
    /// 当句柄需要将参数值传入模板时会被系统调用。
    func extendValuesForResponse(context contxt: MustacheWebEvaluationContext, collector: MustacheEvaluationOutputCollector)
}
```

您在程序中实现的模板页句柄形式，可以参考以下例子：

``` swift
struct TestHandler: MustachePageHandler { // 所有目标句柄都必须从PageHandler对象继承
    // 以下句柄函数必须在程序中实现
    // 当句柄需要将参数值传入模板时会被系统调用。
    // - 参数 context 上下文环境：类型为MustacheWebEvaluationContext，为程序内读取HTTPRequest请求内容而保存的所有信息
    // - 参数 collector 结果搜集器：类型为MustacheEvaluationOutputCollector，用于调整模板输出。比如一个`defaultEncodingFunc`默认编码函数将被安装用于改变输出结果的编码方式。
    func extendValuesForResponse(context contxt: MustacheWebEvaluationContext, collector: MustacheEvaluationOutputCollector) {
        var values = MustacheEvaluationContext.MapType()
        values["value"] = "你好"
        /// 等等等等
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

要重新定向Mustache模板的输出，请调用```mustacheRequest```函数，定义如下：

``` swift
public func mustacheRequest(request req: HTTPRequest, response: HTTPResponse, handler: MustachePageHandler, templatePath: String)
```

该函数参数为当前的HTTP请求与响应对象、您的 ```MustachePageHandler```页面句柄、目标的目标文件路径。```mustacheRequest```会执行初始化操作，比如Mustache模板解析器、定位模板文件，然后调用您的Mustache句柄、填写具体参数值，最终完成页面内容

下面的例子描述了具体如何在URL重定向管理上如何使用Mustache模板。案例中，存储在web根目录下的模板的名字叫“test.html”：

``` swift
{
    request, response in
    let webRoot = request.documentRoot
    mustacheRequest(request: request, response: response, handler: TestHandler(), templatePath: webRoot + "/test.html")
}
```

更详细的例子请参考[多文件上传示例](https://github.com/PerfectExamples/Perfect-UploadEnumerator)便于更好理解本工具集。

### 独立使用（非服务器）方法

Mustache也可以在无服务器的环境下直接使用。只需要提供一个模板文件的路径，或者直接用一个字符串代替模板数据即可。两种情况下模板内容都会被解析并将占位符替换为具体的参数值。

第一个例子使用字符串作为模板，第二个例子则用一个目录文件作为模板：

``` swift
let templateText = "TOP {\n{{#name}}\n{{name}}{{/name}}\n}\nBOTTOM"
let d = ["name":"The name"] as [String:Any]
let context = MustacheEvaluationContext(templateContent: templateText, map: d)
let collector = MustacheEvaluationOutputCollector()
let responseString = try context.formulateResponse(withCollector: collector)
XCTAssertEqual(responseString, "TOP {\n\nThe name\n}\nBOTTOM")
```

``` swift
let templatePath = "path/to/template.mustache"
let d = ["name":"The name"] as [String:Any]
let context = MustacheEvaluationContext(templatePath: templatePath, map: d)
let collector = MustacheEvaluationOutputCollector()
let responseString = try context.formulateResponse(withCollector: collector)
```

### 模板内的变量标签

Mustache模板处理器支持的标签包括：

* {{普通标签}}
* {{{未编码标签}}}
* {{# 区段}} ... {{/区段}}
* {{^ 逆区段}} ... {{/逆区段}}
* {{! 注释}}
* {{> 引用文件}}
* lambda表达式

**引用文件**

所有待引用的文件都必须存储在与模板文件相同的目录下。此外，所有的待引用文件都 *必须* 有 **Mustache** 后缀名。但是这种扩展文件不能在自身内容中再次循环引用自己。比如，为了包含文件 *foo.mustache* 的内容，您可以使用```{{> foo }}```标签。

**编码**

默认情况下，所有被编码标签（也就是普通标签）都是用HTML编码的，而像&lt; &amp; &gt;这样的符号将会被转义处理。在您的句柄中您可以手工设置```MustacheEvaluationOutputCollector.defaultEncodingFunc```函数来设置您需要哪一种编码。比如，当输出JSON数据时您可能需要设置该函数采取如下操作：

``` swift
collector.defaultEncodingFunc = {
    string in
    return (try? string.jsonEncodedString()) ?? "bad string"
}
```

**Lambda表达式**

函数也可以作为参数值加入到对照字典中去。这些函数首先会被执行然后将结果输出到目标模板。这类函数应符合以下方式进行编程：

``` swift
(tag: String, context: MustacheEvaluationContext) -> String
```

`tag` 参数就是标签名称。比如标签`{{name}}`会把变量“name”的值作为标签参数。
