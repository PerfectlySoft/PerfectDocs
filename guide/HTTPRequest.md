# HTTPRequest请求对象

当处理一个HTTP请求时，所有客户端的互动操作都是通过HTTPRequest请求对象和HTTPResponse响应对象实现的。

HTTPRequest对象包含了客户端浏览器发过来的全部数据，包括请求消息头、查询参数、POST表单数据以及其它所有相关信息，比如客户IP地址和URL变量。

HTTPRequest对象将采用`application/x-www-form-urlencoded`编码格式对客户请求进行解析解码。而如果请求中采用`multipart/form-data`“多段”编码方式，则HTTP请求可以把各种未处理的原始格式表单传输过来。当处理“多段”表单数据时，HTTPRequest对象会为请求上传的文件自动创建临时目录并执行解码。这些文件会在请求过程中一直保持直到请求处理完毕，随后自动被删除。

以上涉及到的各种属性和函数都是HTTPRequest请求协议的部分内容。

### Meta Data元数据

HTTPRequest对象还提供一些并非被客户端浏览器显式说明的数据，比如客户端和服务器地址，TCP端口，以及对应分类的文档根目录。

客户端服务器的地址是IP地址和端口的成对信息：

```swift
/// 连接到客户端的IP地址和端口。
var remoteAddress: (host: String, port: UInt16) { get }
/// 服务器方的IP地址和监听端口。
var serverAddress: (host: String, port: UInt16) { get }
```

当服务器创建时，您可以为其设置一个正式的服务器名`CNAME`。很多情况下这个名称非常有用，比如为服务器创建完整的连接。当HTTPRequest创建后，服务器会为其应用`CNAME`，可以从下列属性进行访问：

```swift
/// 服务器正式名称
var serverName: String { get }
```

服务器的文档根目录是为静态文件准备的。如果您不准备提供静态内容，则可以不需要设置文档根目录。文档根目录是在服务器启动之前预先配置好的。当HTTPRequest对象创建时会自动包括这个文档根目录。如果需要访问如Mustache模板这类的静态资源，最好就此准备好文档根目录：

```swift
/// 服务器文档根目录，用于静态文件存储和服务提供。
var documentRoot: String { get }
```

### Request Line请求文本行

HTTPRequest请求文本行包含了请求的方法、路径、查询参数和HTTP协议标识符。典型的HTTPRequest请求文本行形如：

```swift
GET /path?q1=v1&q2=v2 HTTP/1.1
```

HTTPRequest将解析这个请求并转化为以下属性。查询参数`queryParams`是以一个键／值构造的（字典）数组：

```swift
/// The HTTP request method.
var method: HTTPMethod { get set }
/// The request path.
var path: String { get set }
/// The parsed and decoded query/search arguments.
var queryParams: [(String, String)] { get }
/// The HTTP protocol version. For example (1, 0), (1, 1), (2, 0)
var protocolVersion: (Int, Int) { get }
```

在请求响应路由处理过程中，路由URI字串可有多个URL变量组成，可以被解析为一个字典：

```swift
/// 在请求句柄中的URL变量。
var urlVariables: [String:String] { get set }
```

HTTPRequest对象还提供完整的URI请求信息，与其它URL编码查询参数`query parameters`同样方式保存请求路径：

```swift
/// 返回完整的URI唯一资源标识符。
var uri: String { get }
```

### Client Headers客户请求消息头

客户请求的消息头可以用命名访问，或者用遍历方式访问，。HTTPRequest会自动解析所有HTTP cookie的字段名称和字段值。所有可能的请求消息头字段由枚举类型```HTTPRequestHeader.Name```说明，其中还包含了一个可以自定义的```.custom(name: String)```消息头字段

在收到请求后是可以自行设置消息头内容的。在HTTPRequest过滤器中非常有用，因为它们可以重写部分消息头内容：

```swift
/// 返回请求的消息头变量值。
func header(_ named: HTTPRequestHeader.Name) -> String?
/// 为响应增加一个消息头
/// 不会检查目前是否存在重复的消息头。
func addHeader(_ named: HTTPRequestHeader.Name, value: String)
/// 这只消息头变量的属性值。
/// 如果之前消息头已经存在则现有属性值会被替换。
func setHeader(_ named: HTTPRequestHeader.Name, value: String)
/// 遍历所有当前消息头内的属性数据。
var headers: AnyIterator<(HTTPRequestHeader.Name, String)> { get }
```

Cookie可以通过键／值数组进行访问。

```swift
/// 返回目前请求内的所有cookie的键／值。
var cookies: [(String, String)]
```

### GET和POST参数

关于GET和POST参数的详细讨论，详见[使用表单数据](formData.md)。

### 消息体数据

对于“application/x-www-form-urlencoded”和“multipart/form-data”编码类型来说，HTTPRequest对象会通过```postParams```或```postFileUploads```方法自动解码请求内容。

关于文件上传的详细处理方法，请参考[文件上传](fileUploads.md)。关于```postParams```函数的详细使用，请参考[使用表单数据](formData.md)。

其它编码类型的消息体数据不会被解码，只能通过原始字节流或者原始字符串的方式进行访问。比如，对于客户发送的JSON数据，使用字符串的方法会比较有用。

HTTPRequest对象可以通过以下方法访问消息体数据：

```swift
/// 以原始字节流的方式获取POST消息体数据。
/// 如果POST内容类型是multipart/form-data多段表单的话，下面的方法返回为nil。
var postBodyBytes: [UInt8]? { get set }
/// 如果需要的话，POST消息体数据会以UTF-8编码方式解码为一个字符串。
/// 如果POST内容类型是multipart/form-data多段表单的话，下面的方法返回为nil。
var postBodyString: String? { get }
```

**⚠️注意⚠️** 如果请求编码为“multipart/form-data”则```postBodyBytes```属性会为空。否则，该属性总会返回请求的消息体数据，无论请求用哪一种内容类型编码。

注意```postBodyString``` 属性会尝试将数据从UTF-8字节流转化为一个字符串。如果没有消息体数据或者数据无法以UTF-8解码，则返回内容为空。
