# HTTP请求与响应过滤器

除了常规的请求/响应句柄系统之外，Perfect服务器还提供一个请求与响应过滤的系统。增加到服务器上的过滤器在每次客户请求的时候都会被调用。这些过滤器会依次执行，并且有机会要么在提交到处理句柄之前修改，要么在请求后的响应对象中标记为请求完成。过滤器还有权终止当前请求。

过滤器在注册到服务器上时带有一个优先级参数，优先级可以是低、中、高。高优先级的过滤器优先处理，中级次之，低级最次。

因为每次网页发送请求时过滤器都会被激活，因此一定要注意过滤器必须要在最短时间内完成任务，否则会导致响应延迟。

### 请求过滤器

请求过滤器在服务器完全收到请求后会被激活，但是处理完请求过滤器后才会转到指定的网页请求管理句柄。因此过滤器是有机会在由请求句柄处理之前修改请求内容的。

### 创建

请求过滤器必须符合```HTTPRequestFilter```协议

``` swift
/// 允许修改HTTPRequest内容的过滤器
public protocol HTTPRequestFilter {
    /// 在请求完全被服务器接收到后，在转向任何处理器前执行过滤程序。
    func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ())
}
```

当过滤器运行时，其`filter`过滤功能会被调用。过滤器可以执行任何需要的活动然后再调用回调函数通知其过滤工作已结束。回调函数通过读取过滤器状态结果来判断下一步需要做的工作，比如是继续转到其它过滤器、在当前请求过滤优先级内停止过滤、转到消息体控制句柄，还是彻底终止请求。

``` swift
/// 过滤器返回值。
public enum HTTPRequestFilterResult {
    /// 继续过滤。
    case `continue`(HTTPRequest, HTTPResponse)
    /// 终止请求，不调用消息体句柄。
    case halt(HTTPRequest, HTTPResponse)
    /// 停止过滤直接执行请求。
    /// 此时同等优先级的过滤器会被忽略。
    case execute(HTTPRequest, HTTPResponse)
}
```

因为过滤器能够同时管理请求和响应对象，因此在其过滤结果`HTTPRequestFilterResult`中，如果有需要的话，过滤器完全可以彻底替换这些对象。

### 追加过滤器

请求过滤器可以参考以下定义直接追加到服务器上，参数为一个由过滤器与其优先级组成的成对组合：

``` swift
public class HTTPServer {
    public func setRequestFilters(_ request: [(HTTPRequestFilter, HTTPFilterPriority)]) -> HTTPServer
}
```

调用以上方法将设置服务器的所有过滤器。每个过滤器都跟随着该过滤器的优先级。过滤器在数组内没有顺序之分，而服务器会根据优先级自动对这些过滤器进行排序，先高后低。同等优先级的两个过滤器保持原有顺序不变。

### 举例

以下例子是从过滤器有关的测试用例中提取的。该例子显示了如何创建和增加过滤器，以及过滤器优先级是如何交互的。

``` swift
var oneSet = false
var twoSet = false
var threeSet = false

struct Filter1: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		oneSet = true
		callback(.continue(request, response))
	}
}
struct Filter2: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(oneSet)
		XCTAssert(!twoSet && !threeSet)
		twoSet = true
		callback(.execute(request, response))
	}
}
struct Filter3: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(false, "This filter should be skipped")
		callback(.continue(request, response))
	}
}
struct Filter4: HTTPRequestFilter {
	func filter(request: HTTPRequest, response: HTTPResponse, callback: (HTTPRequestFilterResult) -> ()) {
		XCTAssert(oneSet && twoSet)
		XCTAssert(!threeSet)
		threeSet = true
		callback(.halt(request, response))
	}
}

var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
		request, response in
		XCTAssert(false, "This handler should not execute")
		response.completed()
	}
)

let requestFilters: [(HTTPRequestFilter, HTTPFilterPriority)] = [
	(Filter1(), HTTPFilterPriority.high),
	(Filter2(), HTTPFilterPriority.medium),
	(Filter3(), HTTPFilterPriority.medium),
	(Filter4(), HTTPFilterPriority.low)
]

let server = HTTPServer()
server.setRequestFilters(requestFilters)
server.serverPort = 8181
server.addRoutes(routes)
try server.start()

```

### 响应过滤器

每个响应过滤器都是在响应的消息头数据发送给客户端之前执行的。然后才是消息体内的各个数据块传输。这些过滤器可以根据需要修改任何消息体的输出内容，包括增加或删除响应消息头数据，或者覆盖重写消息体内数据。

### 创建

响应过滤器必须符合```HTTPResponseFilter```协议。

``` swift
/// 响应过滤器可以被服务器调用并修改HTTPResponse响应输出
public protocol HTTPResponseFilter {
    /// 在响应消息头发给客户端浏览器之前调用一次。
    func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ())
    /// 可以不调用，也可以被调用多次，来修改发回给客户端浏览器的消息体内容。
    func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ())
}
```

当响应阶段达到发送响应消息头数据时，服务器会调用```filterHeaders```函数。该函数会根据需要调整```HTTPResponse```对象；然后在调用回调函数。调用回调函数时会把过滤器的执行结果```HTTPResponseFilterResult```作为参数传递过去，取值如下：

``` swift
/// 响应过滤器的执行结果。
public enum HTTPResponseFilterResult {
    /// 继续过滤
    case `continue`
    /// 停止执行过滤器直至下一个请求
    case done
    /// 关闭请求
    case halt
}
```

这些取值决定了系统是否应继续处理过滤器、停止执行过滤器直至下一个请求，还是彻底终止并关闭请求。

当准备向客户端浏览器发送数据包时，过滤器的```filterBody```函数会被调用。该函数能够通过```HTTPResponse.bodyBytes```属性检查即将发送的数据，并有机会更改或替换数据内容。因为消息头数据部分已经在这个阶段之前发送走了，任何在过滤器内试图修改消息头数据的操作将会被忽略。一旦过滤器的消息体数据内容完成提交，过滤器会调用回调函数并传递一个```HTTPResponseFilterResult```过滤器结果，该结果类型内容与```filterHeaders```函数中的结果一致。

### 增加过滤器

响应过滤器是直接在服务器上进行设置的，方式是一个由过滤器及其优先级组成的数组。

``` swift
public class HTTPServer {
    public func setResponseFilters(_ response: [(HTTPResponseFilter, HTTPFilterPriority)]) -> HTTPServer
}
```

调用上述函数即完成服务器的响应过滤器的设置。每个过滤器都包含了各自的优先级。数组内的过滤器是没有顺序的。服务器会自动排序，按照优先级从高到低顺序执行。优先级相同的过滤器保持现有顺序不变。

### 举例

以下例子是从过滤器有关的测试用例中提取的。该例子显示了响应过滤器的优先级操作，以及响应过滤器是如何改变目标响应消息头数据和消息体数据内容的。

``` swift
struct Filter1: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		response.setHeader(.custom(name: "X-Custom"), value: "Value")
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
}
struct Filter2: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		var b = response.bodyBytes
		b = b.map { $0 == 65 ? 97 : $0 }
		response.bodyBytes = b
		callback(.continue)
	}
}
struct Filter3: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		var b = response.bodyBytes
		b = b.map { $0 == 66 ? 98 : $0 }
		response.bodyBytes = b
		callback(.done)
	}
}
struct Filter4: HTTPResponseFilter {
	func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		callback(.continue)
	}
	func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
		XCTAssert(false, "This should not execute")
		callback(.done)
	}
}

var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
	request, response in
	response.addHeader(.contentType, value: "text/plain")
	response.isStreaming = true
	response.setBody(string: "ABZ")
	response.push {
		_ in
		response.setBody(string: "ABZ")
		response.completed()
	}
})

let responseFilters: [(HTTPResponseFilter, HTTPFilterPriority)] = [
	(Filter1(), HTTPFilterPriority.high),
	(Filter2(), HTTPFilterPriority.medium),
	(Filter3(), HTTPFilterPriority.low),
	(Filter4(), HTTPFilterPriority.low)
]

let server = HTTPServer()
server.setResponseFilters(responseFilters)
server.serverPort = port
server.addRoutes(routes)
try server.start()
```

在案例中，过滤器增加了一个X-Custom消息头，并将所有消息体数据内的A或B字母全部改成了小写。注意案例中的消息体处理句柄将响应设置为了流模式，意味着大量HTML内容编码工作可以快速执行，而且消息体数据被拆分为两个离散的数据块。

#### 404 响应过滤器

以下的例子很有用。以下代码将创建和安装一个特殊的过滤器用于监控“404 文件未找到”的响应。一旦发生这种情况就用一个自定义的消息代替。

``` swift
struct Filter404: HTTPResponseFilter {
    func filterBody(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
        callback(.continue)
    }

    func filterHeaders(response: HTTPResponse, callback: (HTTPResponseFilterResult) -> ()) {
        if case .notFound = response.status {
            response.setBody(string: "文件 \(response.request.path) 不存在。")
            response.setHeader(.contentLength, value: "\(response.bodyBytes.count)")
            callback(.done)
        } else {
            callback(.continue)
        }
    }
}

let server = HTTPServer()
server.setResponseFilters([(Filter404(), .high)])
server.serverPort = 8181
try server.start()
```
