# Perfect-Mosquitto 

This project provides a Swift class wrapper of `mosquitto` client library which implements the MQTT protocol version 3.1 and 3.1.1.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project, however, it can work independently as a Server Side Swift component.

Ensure you have installed and activated the latest Swift 4.0 tool chain.

## OS X Notes

### Homebrew Installation

This project depends on mosquitto library. To install on mac OS, try command `brew`:

```
$ brew install mosquitto
```

### PC File

A package configuration file is needed, for example, `/usr/local/lib/pkgconfig/mosquitto.pc` as below:

```
Name: mosquitto
Description: Mosquitto Client Library
Version: 1.4.11
Requires:
Libs: -L/usr/local/lib -lmosquitto
Cflags: -I/usr/local/include

```


Please also export an environmental variable called `$PKG_CONFIG_PATH`:

```
$ export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:/usr/lib/pkgconfig"
```

## Linux Notes

This project depends on Ubuntu 16.04 library `libmosquitto-dev`:

```
$ apt-get libmosquitto-dev
```

## Quick Start

### Library Open / Close

Before using any functions of Perfect Mosquitto, please initialize the function set by calling `Mosquitto.OpenLibrary()`. Also, `Mosquitto.CloseLibrary()` is highly recommended once program quitted.

*Note* Both operations are not thread safe, so please perform the operation prior to any threads.

### Initialize a Mosquitto Client

Mosquitto instance can be constructed by calling with or without any parameters. The simplest form can be:

``` swift
let m = Mosquitto()
```

Which means assigning a random client id to this new instance, while all messages and subscriptions will be cleared once disconnected.

However, you can also assign it with a customized client id with instruction of keeping all messages and subscription by this specific name. This is useful for resuming work in case of connection loss.

``` swift
let mosquitto = Mosquitto(id: "myInstanceId", cleanSession: false)
```

### Connect to a Message Broker

A message broker is a server that implements MQTT protocol and serves all clients in terms of messaging - receiving messages from producer and dispatching them to message subscribers.

Although connection to a message broker can be asynchronous, keeping alive or binding to a specific network address, api `connect()` can be as express as demo below - only a host name and a port (usually 1883) are required:

``` swift
try moosquitto.connect(host: "mybroker.com", port: 1883)
```

Although the instance can `disconnect()` from a broker automatically when no longer uses the object, it is recommended to call this function explicitly for a better practice. Besides, a `reconnect()` function is available for the same instance.

### Threading Model

#### Start / Stop
Perfect Mosquitto is flexible in dealing with threads. Clients can call `start()` to run the mosquitto thread in background, which will automatically execute message publishing / receiving without any extra operations in the main thread, i.e., the thread will do the actual sending after calling `publish()` and activate callbacks for incoming messages. If no longer running, you can also stop the service thread at any time by calling `stop()`

``` swift
// start the messaging thread as a background service, it will not block the main thread and will return immediately.
try mosquitto.start()

// do your other work in the main thread, such as publishing etc., and the messages received will go to the callbacks

// stop the background messaging service if no longer need.
try mosquitto.stop()
```

#### Wait for Event
Or, alternatively, you can process events in the main thread, by calling `wait()` method frequently in a message polling style:

``` swift
// wait a minimal while for events. 
// In this specific moment, mosquitto will perform actual message sending,
// and pull messages from broker / run call backs.
try mosquitto.wait(0)
```

The only parameter of `wait()` is the timeout value for message polling in milliseconds. Zero represents the minimal period of the system wait and negative value will be treated the same as 1000 (1 second).

⚠️ *NOTE* ⚠️ *DO NOT MIX `start()`/`stop()` with `wait()`*


### Publish a Message

Once connected to a broker, you can send messages at any time you want:

``` swift
var msg = Mosquitto.Message()
msg.id = 100 // input your message id here
msg.topic = "publish/test"
msg.string = "publication test 🇨🇳🇨🇦"

let mid = try mosquitto.publish(message: msg)
```
As demo above, firstly setup an empty message structure, then assign an message id (integer), a topic for this message and the message content; Then call `publish()` method to send it to the broker with a returned message id. 

*Note* Message content can also be a binary buffer, for example:
``` swift
// send a [Int8] array
msg.payload = [40, 41, 42, 43, 44, 45]
```

Once published, call `start()` or `wait()` to perform the actual message sending as described in the Thread Model.

### Message Subscription and Receiving

The only way to receive a MQTT message in Perfect Mosquitto is messaging callback:

``` swift

mosquitto.OnMessage = { msg in

	// print out message id
	print(msg.id)

	// print out message topic
	print(msg.topic)

	// print out message content
	print(msg.string)

	// print out message body, in a binary array form
	print(msg.payload)
}//end on Message

```

Once set the callback, you can call `subscribe()` to complete the message subscription on client side:

``` swift
try mosquitto.subscribe(topic: "publish/test")
```

Once subscribed, call `start()` or `wait()` to perform actual receiving process as described in the Thread Model.

## More API

Perfect Mosquitto also provides a rich set of functions beside the above ones, please check the project references for detail information.

### Event Callbacks

Please set the following event callbacks for your mosquitto objects if need:

API|Parameters|Description
---|----------|-----------
`OnConnect { status in }` | `ConnectionStatus` | Triggered on connection
`OnDisconnected { status in }` | `ConnectionStatus` | Triggered on disconnection
`OnPublish { msg in }` | `Message` | Triggered when message sent
`OnMessage { msg in }` | `Message` | Triggered on message arrival
`OnSubscribe { id, qos in }` | `(Int32, [Int32])` | Triggered on subscription
`OnUnsubscribe { id in }` | `Int32` (message id) | Triggered on unsubscription
`OnLog { level, content in }` | `(LogLevel, String)` | Triggered on log output

### TLS Configuration

- Set TLS certification file: `func setTLS(caFile: String, caPath: String, certFile: String? = nil, keyFile: String? = nil, keyPass: String? = nil) throws`

- Set TLS verification method: `func setTLS(verify: SSLVerify = .PEER, version: String? = nil, ciphers: String? = nil) throws`

- Set pre shared key: `func setTLS(psk: String, identity: String, ciphers: String? = nil) throws`

### Miscellaneous Functions

- Configure will information for a mosquitto instance: `func setConfigWill(message: Message?) throws`

- Configure username and password for a mosquitton instance: `func login(username: String? = nil, password: String? = nil) throws`

- Set the number of seconds to wait before retrying messages: `func setMessageRetry(max: UInt32 = 20)`

- Set the number of QoS 1 and 2 messages that can be “in flight” at one time: `func setInflightMessages(max: UInt32 = 20) throws`

- Control the behaviour of the client when it has unexpectedly disconnected: `func reconnectSetDelay(delay: UInt32 = 2, delayMax: UInt32 = 10, backOff: Bool = false) throws`

- Reconnecting to a broker after a connection has been lost: `func reconnect(_ asynchronous: Bool = true) throws`

- Reuse an existing mosquitto instance: `func reset(id: String? = nil, cleanSession: Bool = true) throws`

- Set MQTT Version: `func setClientOption(_ version: MQTTVersion = .V31, value: UnsafeMutableRawPointer) throws`

- Explain an exception (in English): `static func Explain(_ fault: Exception) -> String`
 
