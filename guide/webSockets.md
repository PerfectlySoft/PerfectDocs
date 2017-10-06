# WebSockets

WebSockets are designed as full-duplex communication channels over a single TCP connection. The WebSocket protocol facilitates the real-time data transfer from and to a server. This is made possible by providing a standardized way for the server to send content to the browser without being solicited by the client, and allowing for messages to be passed back and forth while keeping the connection open. In this way, a bi-directional (two-way) ongoing conversation can take place between browser and server. The communications are typically done over standard TCP ports, such as 80 or 443.

The WebSocket protocol is currently supported in most major browsers including Google Chrome, Microsoft Edge, Internet Explorer, Firefox, Safari and Opera. WebSockets also require support from web applications on the server.

## Relevant Examples

* [Perfect-Chat-Demo](https://github.com/PerfectExamples/Perfect-Chat-Demo)
* [Perfect-WebSocketsServer](https://github.com/PerfectExamples/Perfect-WebSocketsServer)


## Getting started

Add the WebSocket dependency to your Package.swift file:

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-WebSockets.git", majorVersion: 3)
```

Then import the WebSocket library into your Swift source code as needed:

``` swift
import PerfectWebSockets
```

A typical scenario is communication inside a web page like a chat room where multiple users are interacting in near-real time.

This example sets up a WebSocket service handler interacting on the route `/echo`:

``` swift
var routes = Routes()

routes.add(method: .get, uri: "/echo", handler: {
		request, response in
    // Provide your closure which will return the service handler.
    WebSocketHandler(handlerProducer: {
        (request: HTTPRequest, protocols: [String]) -> WebSocketSessionHandler? in

        // Check to make sure the client is requesting our "echo" service.
        guard protocols.contains("echo") else {
            return nil
        }

        // Return our service handler.
        return EchoHandler()
    }).handleRequest(request: request, response: response)
	}
)
```

## Handling WebSocket Sessions

A WebSocket service handler must implement the `WebSocketSessionHandler` protocol.

This protocol requires the function `handleSession(request: HTTPRequest, socket: WebSocket)`. This function will be called once the WebSocket connection has been established, at which point it is safe to begin reading and writing messages.

The initial `HTTPRequest` object which instigated the session is provided for reference.

Messages are transmitted through the provided WebSocket object.

* Call `WebSocket.sendStringMessage` or `WebSocket.sendBinaryMessage` to send data to the client.
* Call `WebSocket.readStringMessage` or `WebSocket.readBinaryMessage` to read data from the client.

By default, reading will block indefinitely until a message arrives or a network error occurs.

A read timeout can be set with `WebSocket.readTimeoutSeconds`.

Close the session using `WebSocket.close()`.


The example `EchoHandler` consists of the following:

``` swift
class EchoHandler: WebSocketSessionHandler {

	// The name of the super-protocol we implement.
	// This is optional, but it should match whatever the client-side WebSocket is initialized with.
	let socketProtocol: String? = "echo"

	// This function is called by the WebSocketHandler once the connection has been established.
	func handleSession(request: HTTPRequest, socket: WebSocket) {

		// Read a message from the client as a String.
		// Alternatively we could call `WebSocket.readBytesMessage` to get the data as an array of bytes.
		socket.readStringMessage {
			// This callback is provided:
			//	the received data
			//	the message's op-code
			//	a boolean indicating if the message is complete
			// (as opposed to fragmented)
			string, op, fin in

			// The data parameter might be nil here if either a timeout
			// or a network error, such as the client disconnecting, occurred.
			// By default there is no timeout.
			guard let string = string else {
				// This block will be executed if, for example, the browser window is closed.
				socket.close()
				return
			}

			// Print some information to the console for informational purposes.
			print("Read msg: \(string) op: \(op) fin: \(fin)")

			// Echo the data received back to the client.
			// Pass true for final. This will usually be the case, but WebSockets has
			// the concept of fragmented messages.
			// For example, if one were streaming a large file such as a video,
			// one would pass false for final.
			// This indicates to the receiver that there is more data to come in
			// subsequent messages but that all the data is part of the same logical message.
			// In such a scenario one would pass true for final only on the last bit of the video.
			socket.sendStringMessage(string, final: true) {

				// This callback is called once the message has been sent.
				// Recurse to read and echo new message.
				self.handleSession(request, socket: socket)
			}
		}
	}
}
```

## FastCGI Caveat
WebSockets serving is only supported with the stand-alone Perfect HTTP server. At this time, the WebSocket server does not operate with the Perfect FastCGI connector.

## WebSocket Class

### enum OpcodeType
WebSocket messages can be various types: continuation, text, binary, close, ping or pong, or invalid types.

### var readTimeoutSeconds
When trying to read a message from the current socket, this property helps the socket to read a message before timeout. If this property has been set to NetEvent.noTimeout (-1), it will wait infinitely.

### read message
There are two ways of reading messages: text or binary, which differ only in the data type returned, i.e. String and [UInt8] array respectively.

#### read text message:

``` swift
public func readStringMessage(continuation: @escaping (String?, _ opcode: OpcodeType, _ final: Bool) -> ())
```

#### read binary message:

``` swift
public func readBytesMessage(continuation: @escaping ([UInt8]?, _ opcode: OpcodeType, _ final: Bool) -> ())
```

There are three parameters when it calls back:

#### String / [UInt8]
readMessage will deliver the text / binary data sent from the client to your closure by this parameter.

#### opcode
Use opcode if you want more controls in the communication.

#### final
This parameter indicates whether the message is completed or fragmented.

### send message
There are two ways of sending messages: text or binary, which differ only in the data type sent, i.e. String and [UInt8] array respectively.

#### send text message:

``` swift
public func sendStringMessage(string: String, final: Bool, completion: @escaping () -> ())
```

#### send binary message:

``` swift
public func sendBinaryMessage(bytes: [UInt8], final: Bool, completion: @escaping () -> ())
```

Parameter final indicates whether the message is completed or fragmented.

### ping & pong

Perfect WebSocket also provides a convenient way of testing the connection. The ping method starts the test and expects a pong back.

Check out these two methods:

``` swift
/// Send a "pong" message to the client.
public func sendPong(completion: @escaping () -> ())

/// Send a "ping" message to the client.
	/// Expect a "pong" message to follow.
public func sendPing(completion: @escaping () -> ())
```

### func close()
To close the WebSocket connection:

``` swift
socket.close()
```

For a Perfect WebSockets server example, visit the [Perfect-WebSocketsServer](https://github.com/PerfectExamples/Perfect-WebSocketsServer) demo.
