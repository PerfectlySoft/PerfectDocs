# Perfect-Kafka

该项目实现了 librdkafka 的 Swift类函数库

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分，但也可以独立使用。

请确保您已经安装并激活了最新版本的 Swift 4.0 tool chain 工具链。

## MacOS X 安装指南

在使用本函数库之前，请正确安装 librdkafka：

```
$ brew install librdkafka
```

另外注意可能需要在编译之前设定pkg-config路径环境变量：

```
$ export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
```

## Linux 安装指南

在使用本函数库之前，请正确安装 librdkafka-dev：

```
$ sudo apt-get install librdkafka-dev
```

## 快速上手

### Kafka 客户端配置

在开始任何实际的流操作之前，需要正确配置 Kafka 客户端 - 数据生产者 Producer 或者数据消费者 Consumer。
Perfect Kafka 提供两种不同的配置类型，分别是`Kafka.Config()`用于全局配置，而`Kafka.TopicConfig()`用于配置某个特定的主题。

#### 全局配置的初始化

如果希望直接创建一个所有变量都是默认值的配置，请直接调用：

``` swift
let conf = try Kafka.Config()
```

如果希望从现有配置中制作一个副本，则可以这样使用：

``` swift
let conf = try Kafka.Config()
// 该操作会保存现有配置不变，然后在此基础上制作一个副本：
let conf2 = try Kafka.Config(conf)
```

#### 主题配置的初始化

针对某个主题的配置初始化方法与全局配置的方式是一样的：

创建一个默认主题配置：

``` swift
let conf = try Kafka.TopicConfig()
```

或者从当前主题配置基础之上制作一个副本：

``` swift
let conf = try Kafka.TopicConfig()
// 该操作会保存现有配置不变，然后在此基础上制作一个副本：
let conf2 = try Kafka.TopicConfig(conf)
```

### 配置操作：读取和设置

无论主题配置还是全局配置操作方法都是一样的：

#### 查询配置中的所有变量和取值

`Kafka.Config.properties` 和 `Kafka.TopicConfig.properties` 提供了采用字典方式直接访问变量的方法：

``` swift
// 打印一个配置中的所有变量
print(conf.properties)
// 比如，打印结果有可能类似这样：
// ["topic.metadata.refresh.fast.interval.ms": "250",
// "receive.message.max.bytes": "100000000", ...]
```

#### 获取变量值

调用 `get()` 以获取特定变量的取值：

``` swift
let maxBytes = try conf.get("receive.message.max.bytes")
// 默认情况下 maxBytes 值为 "100000000"
```

#### 变量赋值

调用 `set()` 方法保存变量值

``` swift
// 下列操作将限制接收缓冲为1MB
try conf.set("receive.message.max.bytes", "1048576")
```

### 消息生产者 Producer

Perfect-Kafka 提供一个消息生产者 Producer 类用于向服务器发送数据和消息。生产者可以一次发送一条消息，或者一次发送一批消息。消息可以是文本或者二进制字节流。

``` swift
let producer = try Producer("VideoTest")
let brokers = producer.connect(brokers: "host:9092")
if brokers > 0 {
  let _ = try producer.send(message: "你好，世界！")
}
```

在实际发送任何消息之前，需要采取下列步骤以设置客户端到服务器的连接：

#### 初始化生产者实例并设置消息主题

生产者实例化需要设置对应的消息主题。服务器上可以存在同名主题，也可以从客户端创建一个新的主题。

如果服务器上不存在该主题，`Producer()` 会创建一个新的主题，否则会连接到已存在的主题。

比如，以下代码展示了如何创建一个消息生产者并将主题设置为"VideoTest"：

``` swift
let producer = try Producer("VideoTest")
```

#### 连接到服务器（消息掮客）

使用 `connect()`函数可以将客户端（消息生产者或者消费者）连接到一个或多个消息掮客上去，也就是 Kafka 服务器（主机名和端口）。

``` swift
let brokers = producer.connect(brokers: "host1:9092,host2:9092,host3:9092")
```

如果执行成功，则该函数会返回实际连接到的服务器数量。

该函数还包含两个变种以适应不同的参数格式，比如字符串数组：

``` swift
let brokers = producer.connect(brokers: ["host1:9092", "host2:9092", "host3:9092"])
```

或者字典：
``` swift
let brokers = producer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
```

#### 发送消息

Perfect Kafka 允许向服务器（掮客）一次性发送一条或多条消息，可以是文本消息或者二进制字节流数据：

方法|描述|返回值
------|-----------|-------
`send(message: String, key: String? = nil)`|发送一条消息，并且伴随发送该消息的关键词（可选）|64位整型消息编号
`send(message: [Int8], key: [Int8] = [])`|发送一条二进制消息，并且伴随发送该消息的关键词（可选）|64位整型消息编号
`send(messages: [(String, String?)])`|发送一批消息，每个消息都可以伴随返送关键词（可选）|一个64位整型数组，每个元素都是对应消息的编号
`send(messages: [([Int8], [Int8])])`|发送一批二进制消息，每个消息都可以伴随返送关键词（可选）|一个64位整型数组，每个元素都是对应消息的编号

#### 是否发送成功？

Perfect Kafka 的 `send()` 方法是一个异步函数，因此需要更多步骤来判定每个消息发送的状态。

- `OnSent()` 回调函数。 如果该事件被正确设置，则每个消息发送时都会回调这个事件。比如 `producer.OnSent = { print("第 #\($0) 号消息已经发送成功") }`。该事件的唯一参数是由`send()`返回的64位整型消息编号。

- `producer.outbox` 是一个64位整型数组，说明正在发送队列中等待发送的消息。⚠️注意⚠️ 由于Kafka是一个高性能流处理平台，因此这个发件箱内的消息编号并不意味着此时无法发送。所以除非出错，否则尽量避免按照该发件箱的内容进行消息重发。

- `OnError()` 出错回调。消息生产者可以使用该事件处理异常，比如 `producer.OnError = { print("发送错误： \($0)") }` 将会输出具体的错误信息。

- `flush(_ seconds: Int)` 方法可以用于等待一段时间（单位是秒）便于系统将队列中的数据全部发送。

### 消息消费者 Consumer

与消息生产者初始化方法类似，消费者初始化也同样需要设置主题词并连接到具体的服务器：

``` swift
let consumer = try Consumer("VideoTest")
let brokers = consumer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
guard brokers > 0 else {
  // 连接失败
}//end guard
```

#### 消息分区 Partitions

一旦完成连接，最好调用以下方法获取服务器上的消息状况：

``` swift
let info = try consumer.brokerInfo()
print(info)
```

上述示范中的变量 `info`是一个 `MetaData`数据结构，内容展开如下：

成员变量|类型|说明
------|----|-----------
brokers|[Broker]|一个 Broker（消息掮客）结构数组
topics|[Topic]|一个 Topic（主题内容）结构数组

数据结构`Broker`（掮客服务器）保存了以下信息：

成员变量|类型|说明
------|----|-----------
id|Int|掮客服务器代号
host|String|掮客服务器主机名称
port|Int|掮客服务器TCP端口号

数据结构 `Topic`（主题内容）中的核心信息是分区：

成员变量|类型|说明
------|----|-----------
name|String|主题词
err|Exception|该主题是否包含错误
partitions|[Partition]|该主题的分区

数据结构 `Partition`（分区） 对于所有消息操作来说都非常重要：

成员变量|类型|说明
------|----|-----------
id|Int|Partition Id - 分区代码。请使用该代码启动／停止消息处理
err|Exception|分区内是否包含错误
leader|Int|首选掮客服务器（主）
replicas|[Int]|备份掮客服务器（仆）
isrs|[Int]|正在同步过程中的掮客服务器（准仆）

以下代码示范了如何获取分区的关键信息：

``` swift
let consumer = try Consumer("VideoTest")
let brokers = consumer.connect(brokers: ["host1": 9092, "host2": 9092, "host3": 9092])
guard brokers > 0 else {
  // 连接失败
}//end guard
consumer.OnArrival = { m in print("收到消息，编号 #\(m.offset) 内容 \(m.text)")}
let info = try consumer.brokerInfo()
guard info.topics.count > 0 else {
  // 主题内容不存在
}//end guard
guard info.topics[0].name == "VideoTest" else {
  // 主题内容并非预期
}//end guar
let partitions = info.topics[0].partitions
```

#### 从分区上下载数据

以下代码展示了如何从特定分区中下载数据。这个例子中我们假设 `let partId = partitions[0].id`:

``` swift
consumer.OnArrival = { m in
  print("收到消息，编号 #\(m.offset) 内容 \(m.text)")
}//end event

// 开始下载
try consumer.start(partition: partId)
// 坚持下载直到程序退出
while(notEndOfProgram) {
  let total = try consumer.poll(partition: partId)
  print("本时段共下载了 \(total) 条数据")
}//end while
consumer.stop(partId)
```

以下为上述代码的展开解释：

首先，`OnArrival()`事件会提供一个`Message`数据结构：

成员变量|类型|说明
------|----|-----------
err|Exception|本消息包是否有误
topic|String|消息主题词
partition|Int|消息所在分区
isText|Bool|该消息是否为UTF-8有效编码字符串
data|[Int8]|原始的二进制字符流消息数据
text|String|如果`isText`为真，则可以解码的 UTF-8 字符串
keyIsText|Bool|该消息的关键词是否为有效的 UTF-8 字符串
keybuf|[Int8]|可选关键词的二进制字符数组
key|String|如果`keyIsText`为真，则关键词解码为 UTF-8 字符串
offset|Int64|在消息主题内该分区的关键索引

其次，调用`start()`函数下载消息： `func start(_ from: Position = .BEGIN, partition: Int32 = RD_KAFKA_PARTITION_UA)`，以下为详细的参数细节：
- from: Position，即确定在分区中从那一条消息开始下载。允许值包括1、 `.BEGIN`，即从头下载；2、`.END`，只下载最新的内容；3、`.STORED` 从之前保存的断点开始下载；4、`.SPECIFY(Int64)`从某个具体位置开始下载。 ⚠️注意⚠️ 如果需要使用断点续传，则请使用 `func store(_ offset: Int64, partition: Int32 = RD_KAFKA_PARTITION_UA)` 在服务器上保存断点。
- partition: Int32，分区编号

之后可以使用 Perfect Kafka 提供的轮询函数 `poll()`来等待一段时间，监控具体分区的消息活动：
`func poll(_ timeout: UInt = 10, partition: Int32 = RD_KAFKA_PARTITION_UA)`。其中 `timeout` 是轮询等待的毫秒数。

最后，请调用 `stop()` 结束消息处理。
