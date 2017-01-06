# HTTPResponse响应对象

当处理一个HTTP请求时，所有客户端的互动操作都是通过HTTPRequest请求对象和HTTPResponse响应对象实现的。

HTTPResponse响应对象包含了所有即将作为响应发回到客户端浏览器的数据。该对象包含了HTTP状态代码和对应消息，HTTP消息头，以及页面内容数据。HTTPResponse还包含了流数据和浏览器推送的能力，可以控制完成请求或终止请求。

以下内容为HTTPResponse对象协议的属性和方法，非该对象的内容会另行说明。

### HTTP状态

HTTP状态用于说明请求是否成功。以返回错误或者采取其它操作。默认情况下HTTPResponse对象包含一个200 OK状态。这个状态也可以设置为任何一个其它值。HTTP状态代码由`HTTPResponseStatus`枚举管理，也包含了可以自定义的`.custom(code: Int, message: String)`状态

返回状态见如下属性：

``` swift
/// HTTP响应状态
var status: HTTPResponseStatus { get set }
```

### Response响应消息头

响应消息头可以从对象内获取、设置或者遍历。常用的官方消息头命名参见```HTTPResponseHeader.Name```枚举。该枚举还包含了可以自定义消息头名称的```.custom(name: String)```类型。

``` swift
/// 返回响应消息头内容值。
func header(_ named: HTTPResponseHeader.Name) -> String?
/// 在响应内容中增加一条消息头数据。
/// 注意添加操作不会自动检查该条目数据是否已经存在或者重复。
func addHeader(_ named: HTTPResponseHeader.Name, value: String)
/// 设置消息头数据
/// 如果消息头中该数据项已经存在，则已存在的数据值会被覆盖。
func setHeader(_ named: HTTPResponseHeader.Name, value: String)
/// 遍历所有目前对象内的消息头数据。
var headers: AnyIterator<(HTTPResponseHeader.Name, String)> { get }
```

HTTPResponse对象还提供用于设置HTTP cookies（即浏览器内保存临时用户信息的一种方法）的更高级支持，即创建一个cookie对象，然后增加到响应对象中去。

``` swift
/// 以下结构用于设置响应内容中的cookie数据
public struct HTTPCookie {
    /// Cookie 构造函数
    public init(name: String,		// 名称
                value: String,		//	值
                domain: String?,		//	所在域名
                expires: Expiration?,	//	有效期
                path: String?,	//	路径
                secure: Bool?,	//	时间
                httpOnly: Bool?)	//	只限于http协议
}
```

用以下函数可以将Cookie增加到HTTPResponse响应

``` swift
/// 在输出响应时增加一个cookie。
func addCookie(_ cookie: HTTPCookie)
```

当cookie增加到HTTP响应之后会被格式化为“Set-Cookie”消息头

### Body Data消息体数据

响应的当前消息体数据可以通过以下属性管理：

``` swift
/// 待发出给客户端浏览器的消息体数据.
/// 每个数据块送出后都会被清空。
var bodyBytes: [UInt8] { get set }
```

数据可以直接追加到数组中去，或者通过下列函数进行内容追加。下列函数可以完全改变消息体数据内容，或者仅仅是追加二进制无符号整数字节或者字符串。字符串会被转换为UTF-8编码。最后一个函数允许将一个`[String:Any]`字典转化为一个JSON字符串。

``` swift
/// 追加二进制字节流到消息体。
func appendBody(bytes: [UInt8])
/// 向响应输出追加字符串，
/// 所有字符串都会被编码为UTF-8 的 [UInt8]二进制字节数组
func appendBody(string: String)
/// 清除当前消息体，并用新的二进制字节数组内容代替。
func setBody(bytes: [UInt8])
/// 清除当前消息体，并用新的字符串内容代替，
/// 所有字符串都会别编码为UTF-8 的 [UInt8]二进制字节数组
func setBody(string: String)
/// 清除当前消息体，并用JSON字典代替。该JSON字典会被转化为一个JSON字符串，并转化为UTF-8 的 [UInt8]二进制字节数组。
func setBody(json: [String:Any]) throws
```

当对客户端发送响应时，一定要注意必须包括消息头内的响应内容长度字节数。当HTTPResponse响应对象开始将积累的数据发给客户端时，会检查其消息头长度是否设置。如果没有设置，则响应对象会根据```bodyBytes```数组长度进行统计。

以下函数用于推送所有响应消息头和消息体数据：

``` swift
/// 将现有全部消息头和消息体数据推送给客户端浏览器。
/// 可能会一次或多次调用。
func push(callback: (Bool) -> ())
```

通常没有必要调用这个函数，因为系统会在请求完成后自动推送所有响应数据。但是某种情况下可能用户系统直接控制这个过程。

比如，服务器需要发送一个非常大的文件，这种情况下不可能把全部内容读到内存中设置消息体数据。因此首先要将文件长度设置为待响应的消息体长度，然后进行分段读写，即先读出一部分文件数据，以读出来的内容设置消息体，然后重复调用```push```函数直到全部文件内容发送完毕。```push```函数会在完成时调用参数中的callback回调函数，并带一个布尔变量作为调用回调函数的参数。如果内容已经成功发送给客户端浏览器，则该布尔变量会被设置为真值。如果布尔值为假，则意味着该请求失败，也不会需要再采取任何操作。

### 流媒体处理模式

某些情况下，响应内容的长度无法轻易确定。比如，如果是在线的视频流或者音乐流，则这种情况下是无法设置消息头中的内容长度的。此时，请将HTTPResponse设置为流媒体模式，这样响应会以HTTP数据块编码的方式进行发送。流媒体不要求设置响应内容长度单位，相反，只需要对响应增加消息体并调用```push```函数。如果push成功，则消息体会被清空然后可以继续追加更多数据。请持续调用```push```直到请求完成，或者push函数会给回调函数发一个```false```表示响应失败。

``` swift
/// 确定响应切换为流媒体输出模式。
/// 最大的区别就是响应内容长度是未知的。
var isStreaming: Bool { get set }
```

如果确定要使用流媒体模式，则需要在给客户端浏览器推送数据前，将```isStreaming```属性设置为真。

### 完成请求

**⚠️注意⚠️**：当请求完成时，**必须调用** HTTPResponse的```completed```函数。必须确认所有的数据已经完全送达给客户，随后TCP连接会被关闭，无论是不是处于HTTP keep-alive活动状态，直到有新的请求开始处理。

``` swift
/// 意味着请求已经全部完成。
/// 所有未发送的消息头数据和消息体数据此时都会推送给客户。
/// 一旦调用完成则后续操作均失去意义。
func completed()
```
