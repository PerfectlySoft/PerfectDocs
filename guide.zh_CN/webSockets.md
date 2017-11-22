# WebSockets

WebSocket是建立在一个单独的TCP连接基础之上的全双工通讯协议，目前广泛被应用与各类浏览器和Web服务器。WebSocket协议使得浏览器和服务器之间的实时通信和通信连接维持成为可能。通信过程在TCP端口上完成，比如80端口或443端口。

WebSocket协议目前支持大多数主流浏览器，包括谷歌Chrome、微软Edge和IE、火狐、苹果Safari，以及Opera。WebSocket的实现需要网页程序和后台服务器的同时支持。


## 准备开始

请修改您的 Package.swift 文件并增加以下依存关系：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-WebSockets.git", majorVersion: 3)
```

保存好后就可以在您的源程序内使用WebSocket函数库了：

``` swift
import PerfectWebSockets
```

最简单的应用可以在网页中实现，比如聊天室 —— 用户一旦输入了一些内容，其他在同一个网页上的人都能够同步看到这个结果。

以下代码展示了如何在路由`/echo`上增加WebSocket服务具柄：

``` swift
var routes = Routes()

routes.add(method: .get, uri: "/echo", handler: {
		request, response in
    // 具柄创建后，您需要为具柄提供一个闭包便于回调：
    WebSocketHandler(handlerProducer: {
        (request: WebRequest, protocols: [String]) -> WebSocketSessionHandler? in

        // 检查客户端是否在请求我们的“echo“服务。
        guard protocols.contains("echo") else {
            return nil
        }

        // 返回真正的服务具柄
        return EchoHandler()
    }).handleRequest(request: request, response: response)
	}
)
```

## 处理 WebSocket 会话

WebSocket服务具柄必须要实现`WebSocketSessionHandler`会话控制协议。该协议需要一个函数用于处理请求和socket会话：

该函数会在WebSocket连接时调用，随后您就可以开始读写信息了。最开始的 `WebRequest` 对象用于控制会话，而消息通过WebSocket对象进行传输。

首先`WebRequest`对象用于启动会话。

消息则通过 WebSocket 对象返回。

调用 `WebSocket.sendStringMessage` 或 `WebSocket.sendBinaryMessage` 给客户端发数据。
调用 `WebSocket.readStringMessage` 或 `WebSocket.readBinaryMessage` 则用于接收从客户端发来的数据。

默认情况下，读操作会造成程序阻塞直到有消息到达或者网络错误发生。

设置读操作超时的方法是设置`WebSocket.readTimeoutSeconds`属性。

当会话结束则调用`WebSocket.close()`方法关闭连接。


请参考以下例子——在WebSocketSessionHandler会话控制具柄上扩展出的“回声”实例——运行这个例子，客户端socket发出的内容就会被原封不动地返回。

``` swift
class EchoHandler: WebSocketSessionHandler {

	// 超级协议名称，我们自己定义的
	// 虽然是可选项，但是应该与客户端WebSocket创建的协议名称完全一致。
	let socketProtocol: String? = "echo"

	// 该函数在 WebSocketHandler 一旦建立连接后就会被调用。
	func handleSession(request: WebRequest, socket: WebSocket) {

		// 从客户端读取文字消息
		// 如果要读取二进制数据则用 `WebSocket.readBytesMessage` 方法
		socket.readStringMessage {
			// 回调函数的参数包括：
			//	string: 接收到的数据
			//	op：该消息的操作代码
			//	fin：数据已经读取完毕，如果是真就是读完了。
			//  （反之意味着数据只是被部分读取）
			string, op, fin in

			// 如果接收到的数据为空，那么有可能是超时，
			// 也有可能是因为网络异常，比如客户端掉线等等
			// 默认情况下是不会超时的。
			guard let string = string else {
				// 如果客户端窗口关闭这类的事件发生了，那么就应该在服务器端进行相应的关闭操作。
				socket.close()
				return
			}

			// 输出读取到的内容。
			print("已经获取了信息：\(string) \t操作码：\(op) \t是否全部读取：\(fin)")

			// 将收到的数据原封不动地返回给客户
			// 这里简单起见直接将final设置为真，意味着一次性把数据发完。
			// 但是WebSockets还可以选择一次只发送一个数据片段
			// 比如，如果传输一个大文件，比如视频文件，
			// 那么需要把数据分段发出，分段过程中final值为假。
			// 表明后续数据需要继续发出
			// 而后续内容将与已经发出的数据构成一个逻辑上的完整消息。
			// 如果是发送视频，那么必须是在最后一段数据发完时用上将这个标志变量设置为真。
			socket.sendStringMessage(string, final: true) {

				// 该回调函数会在消息发送完成后调用
				// 递归调用，继续下一个读取操作 —— 一直读下去
				self.handleSession(request, socket: socket)
			}
		}
	}
}
```

## FastCGI 使用注意
WebSockets 目前支持独立运行的 Perfect HTTP 服务器，暂时无法支持 Perfect FastCGI 连接的集成

## WebSocket 类

### enum OpcodeType 操作代码
WebSocket 消息可以有下列几种类型：continuation（连续）、text（文本）、binary（二进制）、close（关闭）、ping or pong（乒乓测试），或者invalid（无效的类型）。

### var readTimeoutSeconds 超时控制
如果要从当前socket 进行读取数据的操作，那么设置这个属性作为读操作的时间限制。如果属性被设置为NetEvent.noTimeout (-1)，则程序会被阻塞并一直等待，不会发生超时的事件。

### read message 读取消息
WebSocket有两种消息格式：文本消息或二进制消息。文本消息数据形式为字符串，而二进制消息数据格式为字节数组[UInt8]。

#### 读取文本消息

``` swift
public func readStringMessage(continuation: @escaping (String?, _ opcode: OpcodeType, _ final: Bool) -> ())
```

#### 读取二进制消息

``` swift
public func readBytesMessage(continuation: @escaping ([UInt8]?, _ opcode: OpcodeType, _ final: Bool) -> ())
```

上述两个方法的参数包括：

#### String / [UInt8]
收到的数据，文本或者二进制数组

#### opcode
操作代码。您可以参考这个代码用于更多更复杂的通信控制。

#### final
读取完成。该参数如果是真，表示数据已经全部收到，假表示部分收到（后面还有数据要发过来）。

### send message 发送消息
同样也有两种方式发送消息：文本消息和二进制消息，区别就在于数据类型，分别是字符串和二进制数组。

#### 发送文本消息

``` swift
public func sendStringMessage(string: String, final: Bool, completion: @escaping () -> ())
```

#### 发送二进制消息

``` swift
public func sendBinaryMessage(bytes: [UInt8], final: Bool, completion: @escaping () -> ())
```

最后一个final参数用于表明该消息是全部还是部分，如果final为假则表示逻辑上还有后续消息会发给客户端，与之前已经发出的消息构成了一个逻辑整体，内容不可分割。

### ping & pong 乒乓测试

Perfect WebSocket 函数库还提供了测试连接的简便函数，即乒乓函数。乒用于发起测试，乓则是在收到乒后返回一个应答。

下面是两个函数的定义：

``` swift
/// 向客户发送乓消息：
public func sendPong(completion: @escaping () -> ())

/// 向客户发送乒消息
/// 并等待乓返回
public func sendPing(completion: @escaping () -> ())
```

### func close() 关闭连接
所有读写操作完毕并不希望继续保持数据连接的情况下，请关闭WebSocket连接。

``` swift
socket.close()
```

详细的WebSockets例子，请参见 [Perfect-WebSocketsServer](https://github.com/PerfectExamples/Perfect-WebSocketsServer)
