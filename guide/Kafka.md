# Perfect-Kafka 


This project provides an express Swift wrapper of librdkafka.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project but can also be used as an independent module.

## Release Notes for MacOS X

Before importing this library, please install librdkafka first:

```
$ brew install librdkafka
```

Please also note that a proper pkg-config path setting is required:

```
$ export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
```

## Release Notes for Linux

Before importing this library, please install librdkafka-dev first:

```
$ sudo apt-get install librdkafka-dev
```

## Quick Start

### Kafka Client Configurations

Before starting any stream operations, it is necessary to apply settings to clients, i.e., producers or consumers.

Perfect Kafka provides two different categories of configuration, i.e. `Kafka.Config()` for global configurations and `Kafka.TopicConfig()` for topic configurations.

#### Initialization of Global Configurations

To create a configuration set with default value settings, simple call:

``` swift
let conf = try Kafka.Config()
```

or, if another configuration based on an existing one can be also duplicated in such a form:

``` swift
let conf = try Kafka.Config()
// this will keep the original settings and duplicate a new one
let conf2 = try Kafka.Config(conf)
```

#### Initialization of Topic Configurations

Topic configuration shares the same initialization fashion with global configuration.

To create a topic configuration with default settings, call:

``` swift
let conf = try Kafka.TopicConfig()
```

or, if another configuration based on an existing one can be also duplicated in such a form:

``` swift
let conf = try Kafka.TopicConfig()
// this will keep the original settings and duplicate a new one
let conf2 = try Kafka.TopicConfig(conf)
```

### Access Settings of Configuration

Both `Kafka.Config` and `Kafka.TopicConfig` have the same api of accessing settings.

#### List All Variables with Value

`Kafka.Config.properties` and `Kafka.TopicConfig.properties` provides dictionary type settings:

``` swift
// this will print out all variables in a configuration
print(conf.properties)
// for example, it will print out something like:
// ["topic.metadata.refresh.fast.interval.ms": "250",
// "receive.message.max.bytes": "100000000", ...]
```

#### Get a Variable Value

Call `get()` to retrieve the value from a specific variable:

``` swift
let maxBytes = try conf.get("receive.message.max.bytes")
// maxBytes would be "100000000" by default
```

#### Set a Variable with New Value

Call `set()` to save settings for a specific variable:

``` swift
// this will restrict message receiving buffer to 1MB
try conf.set("receive.message.max.bytes", "1048576")
```

### Producer

Perfect-Kafka provides a Producer class to send data / message to Kafka hosts. Producer can send a message one at a time, or sent multiple messages in a batch. Messages can be either text string or binary bytes.

``` swift
let producer = try Producer("VideoTest")
let brokers = producer.connect(brokers: "host:9092")
if brokers > 0 {
  let _ = try producer.send(message: "hello, world!")
}
```

Before sending any actual messages, a few steps are required to setup the connection to Kafka hosts.

#### Producer Instance with a Topic

To initialize a Producer instance, a topic name is required no matter whether this topic exists in the Kafka hosts or not.

If the topic didn't exist when connected to Kafka hosts / brokers, `Producer()` would try to create a new one; Otherwise it would use the existing topic for further operations.

For example, the demo below shows how to start a producer with a topic named "VideoTest":

``` swift
let producer = try Producer("VideoTest")
```

#### Connect to Brokers

Use method `connect()` to connect to one or more message brokers, i.e., Kafka hosts ( host and port ):

``` swift
let brokers = producer.connect(brokers: "host1:9092,host2:9092,host3:9092")
```

If success, it will return the number of hosts that connected.

Alternatively, it is also possible to connect to brokers by different parameter fashions, take example, hosts can be an array of string:

``` swift
let brokers = producer.connect(brokers: ["host1:9092", "host2:9092", "host3:9092"])
```

or dictionary:
``` swift
let brokers = producer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
```

#### Send Messages

Perfect Kafka allows to send either text or binary messages to brokers one at a time or in a batch.

Method|Description|Returns
------|-----------|-------
`send(message: String, key: String? = nil)`|a text message with an optional key to send|an Int64 message id
`send(message: [Int8], key: [Int8] = [])`|a binary message with an optional key to send|an Int64 message id
`send(messages: [(String, String?)])`|text messages with optional keys in an array|[Int64] message IDs for each message
`send(messages: [([Int8], [Int8])])`|binary messages with optional keys in an array|[Int64] message IDs for each message

#### Sent or Not

Perfect Kafka `send()` is asynchronous function so the library provides a few extra methods to determine the sending status of each message.

- `OnSent()` callback. If set properly, each message will call this event once actually sent. For example: `producer.OnSent = { print("msg #\($0) was sent") }`. The only parameter of this event is the Int64 message id returned by `send()`.

- `producer.outbox` is an [Int64] array to indicate the messages in sending queue. *NOTE* As a high performant streaming platform, the existence of messages in outbox doesn't mean that they were failed to send, so don't try to resend these message unless it was explicitly confirmed that they were failed to send.

- `OnError()` callback. Producer will call this event if something wrong, e.g., `producer.OnError = { print("error: \($0)") }` will print out the error message if happen.

- `flush(_ seconds: Int)` method can help wait seconds for clearing the message queue and flushing the outbox.

### Consumer

Before actually receiving messages from Kafka with a specific topic, a few procedures are required to initialize a Consumer instance:

``` swift
let consumer = try Consumer("VideoTest")
let brokers = consumer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
guard brokers > 0 else {
  // connection failed
}//end guard
```

#### Partitions

Once connected, it is a good idea to get the information from the brokers to see if there are sufficient resources, i.e., partitions, for further operations:

``` swift
let info = try consumer.brokerInfo()
print(info)
```

The above variable `info` is a `MetaData` structure as reference below:

Member|Type|Description
------|----|-----------
brokers|[Broker]|An array of Broker structure
topics|[Topic]|An array of Topic structure

Structure `Broker` stores the information of a broker:

Member|Type|Description
------|----|-----------
id|Int|Broker Id
host|String|Host name of the broker
port|Int|Host port that listens

The major content of `Topic` structure is to record how many partitions are using in such a topic:

Member|Type|Description
------|----|-----------
name|String|Topic name
err|Exception|Topic error reported by broker
partitions|[Partition]|Partitions of this topic

Data structure `Partition` is vitally important to indicate the partition id for messaging:

Member|Type|Description
------|----|-----------
id|Int|Partition Id - use this to start / stop messaging
err|Exception|Partition error reported by broker
leader|Int|Leader broker
replicas|[Int]|Replica brokers
isrs|[Int]|In-Sync-Replica brokers

Practically, partition info could be acquired by way below:

``` swift
let consumer = try Consumer("VideoTest")
let brokers = consumer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
guard brokers > 0 else {
  // connection failed
}//end guard
consumer.OnArrival = { m in print("message : #\(m.offset) \(m.text)")}
let info = try consumer.brokerInfo()
guard info.topics.count > 0 else {
  // no topic found
}//end guard
guard info.topics[0].name == "VideoTest" else {
  // it is not the topic we want
}//end guar
let partitions = info.topics[0].partitions
```

#### Download Messages From A Partition

Code below shows how to download messages from a partition. In this demo, we assume `let partId = partitions[0].id`:

``` swift
consumer.OnArrival = { m in
  print("message #\(m.offset) : \(m.text)")
}//end event

// start downloading
try consumer.start(partition: partId)
// run until end of program
while(notEndOfProgram) {
  let total = try consumer.poll(partition: partId)
  print("\(total) messages arrived in this moment")
}//end while
consumer.stop(partId)
```

Now we take a walk through:

Firstly, `OnArrival()` event is a callback with a `Message` data structure:
Member|Type|Description
------|----|-----------
err|Exception|Error: if the message is good or not
topic|String|topic name of the message
partition|Int|partition of the message
isText|Bool|if the message is a valid UTF-8 text or not
data|[Int8]|the original binary data of message body
text|String|decoded message body in a UTF-8 string, if `isText`
keyIsText|Bool|if the key is a valid UTF-8 text
keybuf|[Int8]|the original binary data of optional key
key|String|decoded key in a UTF-8 string, if `keyIsText`
offset|Int64|offset inside the topic

Secondly, call `start()` to start download messages: `func start(_ from: Position = .BEGIN, partition: Int32 = RD_KAFKA_PARTITION_UA)`, here are the parameter details:
- from: Position, from which position of the messages in the partition to download. Valid value can be `.BEGIN` to indicate downloading every messages from the very beginning, or `.END` to download the most reason one, or `.STORED` to download the previous stored messages in case of failure, or `.SPECIFY(Int64)` to start downloading from a specific location. *NOTE* use `func store(_ offset: Int64, partition: Int32 = RD_KAFKA_PARTITION_UA)` to store a specific message if `.STORED` is needed.
- partition: Int32, the partition id.

Then Perfect Kafka provides the `poll()` function to wait a short while to listen the activity of a specific partition:
`func poll(_ timeout: UInt = 10, partition: Int32 = RD_KAFKA_PARTITION_UA)`. The `timeout` is the milliseconds to wait for polling.

Finally, call `stop()` to end the messaging.
