# WebSockets

WebSocket is designed as full-duplex communication channels over a single TCP connection, and has already been implemented in a lot of popular web browsers and web servers. The WebSocket protocol makes more interaction between a browser and a web server possible, facilitating the real-time data transfer from and to the server. This is made possible by providing a standardized way for the server to send content to the browser without being solicited by the client, and allowing for messages to be passed back and forth while keeping the connection open. In this way, a two-way (bi-directional) ongoing conversation can take place between a browser and the server. The communications are done over TCP port number 80, which is of benefit for those environments which block non-web Internet connections using a firewall.

The WebSocket protocol is currently supported in most major browsers including Google Chrome, Microsoft Edge, Internet Explorer, Firefox, Safari and Opera. WebSocket also requires web applications on the server to support it.

## System Requirements

Considering that WebSocket is based on a HTTP server, so the PerfectTemplate HTTP server is an essential. If you have no idea about what PerfectTemplate is, please check it out first: [PerfectTemplate HTTP Server](https://github.com/PerfectlySoft/PerfectTemplate/), or at least read this Perfect document first: [Getting Started: PerfectTemplate](gettingStarted.md).


## Import WebSocket Library

Once downloaded & installed PerfectTemplate as the guiding steps, please add a WebSocket dependency to your Package.swift file:

```swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-WebSockets.git", majorVersion: 2, minor: 0)
```

Then you can import WebSocket library into your swift source code as need:

```
import PerfectWebSockets
```

## Quick Start From An Example

Instead of PerfectTemplate, an alternative way to start a project based on a WebSocket framework is [Perfect WebSocket Example](https://github.com/PerfectlySoft/PerfectExample-WebSocketsServer)

## Basic idea

Although WebSocket can perform quite a few complicated operations, the most typical scenario is still a communication inside a web page, for example, a chat room - one user typed something in a page, or an operation such as moving a chess, then all other users in the same page are supposed to see the same outcome.

So here is the idea: set up a route / webpage inside the server first, then manipulate WebSocket handles in this page.

Here is the typical way of setting up a route:

```swift
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
		request, response in
		response.setHeader(.contentType, value: "text/html")
		response.appendBody(string: "<HTML><HEAD><TITLE>Hello</TITLE><body>Hello, world!</body></html>")
    // ... you can append more web contents here
		response.completed()
	}
)
```

Now we can setup a WebSocket service handler to deal with a specific route instead of a static page response:

```swift
routes.add(method: .get, uri: "/echo", handler: {
		request, response in
    // To add a WebSocket service, set the handler to WebSocketHandler.
    // Provide your closure which will return your service handler.
    WebSocketHandler(handlerProducer: {
        (request: WebRequest, protocols: [String]) -> WebSocketSessionHandler? in

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

This protocol requires the function `handleSession(request: WebRequest, socket: WebSocket)`.

This function will be called once the WebSocket connection has been established, at which point it is safe to begin reading and writing messages.

The initial `WebRequest` object which instigated the session is provided for reference.

Messages are transmitted through the provided WebSocket object.

Call `WebSocket.sendStringMessage` or `WebSocket.sendBinaryMessage` to send data to the client.

Call `WebSocket.readStringMessage` or `WebSocket.readBinaryMessage` to read data from the client.

By default, reading will block indefinitely until a message arrives or a network error occurs.

A read timeout can be set with `WebSocket.readTimeoutSeconds`.

When the session is over call `WebSocket.close()`.


The example `EchoHandler` consists of the following.

```swift
class EchoHandler: WebSocketSessionHandler {

	// The name of the super-protocol we implement.
	// This is optional, but it should match whatever the client-side WebSocket is initialized with.
	let socketProtocol: String? = "echo"

	// This function is called by the WebSocketHandler once the connection has been established.
	func handleSession(request: WebRequest, socket: WebSocket) {

		// Read a message from the client as a String.
		// Alternatively we could call `WebSocket.readBytesMessage` to get the data as a String.
		socket.readStringMessage {
			// This callback is provided:
			//	the received data
			//	the message's op-code
			//	a boolean indicating if the message is complete (as opposed to fragmented)
			string, op, fin in

			// The data parameter might be nil here if either a timeout or a network error, such as the client disconnecting, occurred.
			// By default there is no timeout.
			guard let string = string else {
				// This block will be executed if, for example, the browser window is closed.
				socket.close()
				return
			}

			// Print some information to the console for informational purposes.
			print("Read msg: \(string) op: \(op) fin: \(fin)")

			// Echo the data we received back to the client.
			// Pass true for final. This will usually be the case, but WebSockets has the concept of fragmented messages.
			// For example, if one were streaming a large file such as a video, one would pass false for final.
			// This indicates to the receiver that there is more data to come in subsequent messages but that all the data is part of the same logical message.
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
WebSockets serving is only supported with the stand-alone Perfect HTTP server. At this time, the WebSocket server does not operate with the Perfect FastCGI server.

## WebSocket Class

### enum OpcodeType
WebSocket messages can be various types: continuation, text, binary, close, ping or pong, or invalid types.

### var readTimeoutSeconds
When trying to read a message from the current socket, this property helps the socket to read a message before timeout. If this property has been set to NetEvent.noTimeout (-1), it will wait infinitely.

### read message
There are two ways of reading messages: text or binary, and the only difference between both methods is the data returned, i.e, String and [UInt8] array correspondingly.

#### read text message:

```swift

public func readStringMessage(continuation: @escaping (String?, _ opcode: OpcodeType, _ final: Bool) -> ())

```

#### read binary message:

```swift
public func readBytesMessage(continuation: @escaping ([UInt8]?, _ opcode: OpcodeType, _ final: Bool) -> ())

```

There are three parameters when it is calling back:

#### String / [UInt8]
readMessage will deliver the text / binary data sent from client to your closure by this parameter.

#### opcode
Use opcode if you want more controls in the communication.

#### final
This parameter indicates whether the message is a completed or a fragmented.

### send message
There are two ways of sending messages: text or binary, and the only difference between both methods is the data returned, i.e, String and [UInt8] array correspondingly.

#### send text message:

```swift
public func sendStringMessage(string: String, final: Bool, completion: @escaping () -> ())

```

#### send binary message:

```swift
public func sendBinaryMessage(bytes: [UInt8], final: Bool, completion: @escaping () -> ())

```

Parameter final indicates whether the message is a completed or a fragmented.

### ping & pong

Perfect WebSocket also provides a convenient way of testing the connection. The ping method starts the test and expects a pong back.

Check out these two methods:

```swift
/// Send a "pong" message to the client.
public func sendPong(completion: @escaping () -> ())

/// Send a "ping" message to the client.
	/// Expect a "pong" message to follow.
public func sendPing(completion: @escaping () -> ())

```

### func close()
Once everything done, please close the WebSocket connection in this elegant manner.
