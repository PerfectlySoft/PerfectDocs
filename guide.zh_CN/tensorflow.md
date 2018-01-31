# Perfect TensorFlow

本项目为TensorFlow的C语言接口试验性封装函数库，用于Swift在人工智能深度学习上的应用。

本项目需要使用SPM软件包管理器编译并是[Perfect项目](https://github.com/PerfectlySoft/Perfect)的一个组成部分，但也可以独立使用。

请确保您的系统已经安装了Swift 4.0。

## 关于开发

以下文件构成了Perfect-TensorFlow核心：

```
Sources
├── PerfectTensorFlow
│   ├── APILoader.swift (886 行代码，直接从tensorflow/c/c_api.h翻译而来)
│   ├── PerfectTensorFlow.swift (2277 行代码)
└── TensorFlowAPI
    ├── TensorFlowAPI.c (72 行代码)
    └── include
        └── TensorFlowAPI.h (137 行代码)
```

所有以`pb.*.swift`命名的文件（总共目前超过四万五千行）都是从根目录下的 `updateprotos.sh` 文件自动创建的。很不幸的是，用了这个脚本之后，您仍然需要手工编辑 **PerfectTensorFlow.swift** 中的 `public typealias`部分以保持编译一致。

迄今为止暂时没有计划在Swift源代码中创建这些源文件，原因是因为Perfect-TensorFlow是Perfect软件框架体系组成部分之一，虽然可以独立使用，但是也必须符合Perfect的SPM软件包管理器编译标准。尽管如此，我们当然欢迎各类项目合并更新申请、各类意见和建议！

## 快速上手

### 安装

Perfect-TensorFlow 是基于其C语言函数库基础上的，简单说来就是您的计算机上在运行时必须安装 `libtensorflow.so`动态链接库。

本项目包含了一个用于快速安装该链接库 CPU v1.1.0版本的脚本，默认安装路径为`/usr/local/lib/libtensorflow.so`。您可以根据平台要求下载并运行 [`install.sh`](https://github.com/PerfectlySoft/Perfect-TensorFlow/blob/master/install.sh)。运行该脚本之前请确定 `wget`已经安装到您的计算机上；如果还没装，请首先用`brew install wget`（适用于macOS）或者 `sudo apt-get install wget` （适用于Ubuntu）命令安装`wget`组件。


更多的安装选项，如需要在同一台计算机上同时安装CPU/GPU或者多个不同版本，请参考官网网站： [Installing TensorFlow for C](https://www.tensorflow.org/install/install_c)

### Perfect TensorFlow 应用程序

使用之前请在您的项目Package.swift文件中增加依存关系并选择**最新版本**：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-TensorFlow.git", majorVersion: 1)
```

然后声明函数库：

``` swift
// TensorFlowAPI 就是定义在 libtensorflow.so的部分函数集
import TensorFlowAPI

// 这是我们主要介绍的TensorFlow对象封装库
import PerfectTensorFlow

// 为了保持与其他语言函数库版本（比如Python或者Java）的命名规范一致性，
// 为TensorFlow对象取一个缩写名称是个好主意：
public typealias TF = TensorFlow
```

### 激活函数库

⚠️注意⚠️ 在使用  Perfect TensorFlow 的 **任何具体函数之前**，必须首先调用`TF.Open()`方法：

``` swift
// 这个操作会打开 /usr/local/lib/libtensorflow.so 动态链接库
try TF.Open()
```

另外，您还可以激活其他不同规格（CPU/GPU）版本的函数库，所需要的操作就是输入目标函数库路径：
``` swift
// 以下操作将打开非默认路径下的函数库：
try TF.Open("/path/to/DLL/of/libtensorflow.so")
```

### "你好，Perfect TensorFlow!"

以下是 Swift 版本的 "你好, TensorFlow!":

``` swift
// 定义一个字符串型张量：
let tensor = try TF.Tensor.Scalar("你好，Perfect TensorFlow! 🇨🇳🇨🇦")

// 声明一个流程图
let g = try TF.Graph()

// 将张量节点加入流程图
let op = try g.const(tensor: tensor, name: "hello")

// 根据流程图生成会话并运行
let o = try g.runner().fetch(op).addTarget(op).run()

// 解码
let decoded = try TF.Decode(strings: o[0].data, count: 1)

// 检查结果
let s2 = decoded[0].string
print(s2)
```

### 矩阵操作

您可以注意到，其实Swift版本的TensorFlow与其原版内容的概念都是完全一致的，比如创建张量节点，保存节点到流程图、定义操作并运行会话、最后检查结果。

以下是使用Perfect TensorFlow进行矩阵操作的例子：

``` swift
/* 矩阵乘法
| 1 2 |   |0 1|   |0 1|
| 3 4 | * |0 0| = |0 3|
*/
// 输入矩阵
let tA = try TF.Tensor.Matrix([[1, 2], [3, 4]])
let tB = try TF.Tensor.Matrix([[0, 0], [1, 0]])

// 将张量转化为流程图节点
let g = try TF.Graph()
let A = try g.const(tensor: tA, name: "Const_0")
let B = try g.const(tensor: tB, name: "Const_1")

// 定义矩阵乘法操作，即 v = A x Bt，B矩阵的转置
let v = try g.matMul(l: A, r: B, name: "v", transposeB: true)

// 运行会话
let o = try g.runner().fetch(v).addTarget(v).run()
let m:[Float] = try o[0].asArray()
print(m)
// m 的值应该是 [0, 1, 0, 3]
```

### 动态加载已保存的人工神经网络模型

除了动态建立流程图和会话方法之外，Perfect TensorFlow 还提供了将预先保存的模型在运行时加载的简单方法，即从文件中直接还原会话：

``` swift
let g = try TF.Graph()

// 读取模型的签名信息
let metaBuf = try TF.Buffer()

// 还原会话
let session = try g.load(
	exportDir: "/path/to/saved/model",
	tags: ["tag1", "tag2", ...],
	metaGraphDef: metaBuf)
```

### 机器视觉服务器展示

您可以参考下列网址获得Perfect TensorFlow在人工智能机器视觉服务器上的应用：[Perfect TensorFlow Demo](https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision)，在这个服务器上您可以上传或者手绘任何一个图片来测试服务器是否认得这个物品：

<img src='https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision/blob/master/scrshot1.png?raw=true'></img>
<img src='https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision/blob/master/scrshot2.png?raw=true'></img>


# 接口函数指南

Perfect TensorFlow 是在 TensorFlow C语言函数库基础上的 Swift 对象类封装，用于在服务器端开发Swift在TensorFlow方面的应用，能够存储、读取TensorFlow模型。

⚠️**请注意**⚠️：当前本函数库仍然处于试验状态，因此不在TensorFlow函数调用版本稳定性承诺范围内。

目前 [Perfect TensorFlow 机器视觉演示程序](https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision) 展示了本函数方法，具体内容就是加载了一个预先训练好的卷积神经网络模型，并演示：

Graph 构造张量运算图：使用 OperationBuilder 对象构造张量运算图，用于解码、尺寸变换和正规化一个JPEG照片文件。
模型加载：调用Graph.import()方法加载人工智能图像识别模型。
张量运算：使用Session会话对象按图执行张量计算并分析最接近当前图片内容的标签名称。

使用之前，请首先 [安装 Perfect TensorFlow 函数库](#安装) 。

## 对象类简介

Perfect-TensorFlow 函数库是由一系列张量计算有关的对象类构成；为实现完整的编程和运算，请参考下图用于理解各个对象之间的关系和编程顺序：

```
TFLib: TensorFlow C 语言接口底层函数库
  |
TensorFlow: 运行函数库
  |
  |------ Shape: 张量的维度信息
  |         |
  |       Tensor: 多维数组构成的张量
  |         |
 Graph -- OperationBuilder: 用于构造张量运算表达式的辅助类
  |         |
  |       Operation: 运算操作——张量流程图中的一个节点，用于表达张量之间的运算方法
  |         |
  |       Output: 从运算操作转化而来的符号句柄
  |         |
  |------ Session: 执行张量运算的驱动程序
            |
          Runner: 执行具体的张量计算并生成新创建的张量结果
```

### 目录

|名称|描述|
|----------|------------|
|TFLib|TensorFlow C 语言接口函数库，建议仅限程序底层内部使用|
|[TensorFlow](#tensorflow-runtime)|封装TensorFlow各对象类的命名空间|
|[Shape](#shape)|张量的维度对象；直接从一个64位整型数组创建而来|
|[OperationBuilder](#operation-and-its-builder)|用于构造张量运算表达式的辅助类|
|[Operation](#operation-and-its-builder)|运算操作是张量计算图中的一个节点，用于表达张量之间的运算方法|
|[Output](#output)|从运算操作转化而来的符号句柄|
|[Graph](#graph)|用于代表所有张量计算过程的一个流程图|
|[Session Runner](#session)|执行具体的张量计算并生成新创建的张量结果|

### Protocol Buffers 谷歌缓冲协议

Perfect TensorFlow 高度依赖于谷歌的缓冲区协议标准，也就是说您可以非常方便的将大多数函数库内的对象实例保存到二进制文件，或者从二进制文件中加载为目标的对象类实例，并且也可以随时按需转化为数据库记录或者JSON字符串。

以张量运算流程图对象Graph为例，您可以从任何一个训练好的模型文件导入到您的Swift程序中，如以下代码所示[（代码节选自Perfect TensorFlow 机器视觉演示程序）](https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision/blob/master/Sources/main.swift)：

``` swift
import PerfectLib
import PerfectTensorFlow

typealias TF = TensorFlow

// 加载一个第三方张量运算流程图文件：
let fModel = File("/tmp/tensorflow_inception_graph.pb")
try fModel.open(.read)
let modelBytes = try fModel.readSomeBytes(count: fModel.size)
fModel.close()

// 将数据导入为运算流程图对象：
let def = try TF.GraphDef(Data(bytes: modelBytes))
let g = try TF.Graph()
try g.import(definition: def)
```

⚠️**注意**⚠️ 反过来您可以从当前对象中获得缓冲协议数据，具体操作是 `let data = try def.serializedData()`生成二进制数据，或者`let json = try def.jsonString()`将对象转换为JSON字符串。


## TensorFlow Runtime

TensorFlow 类用于代表其运行时的对象命名空间：

为了与其他语言开发的TensowFlow函数库保持一致，比如Java或者Python版本，此处建议增加`TensorFlow`类的别名：

``` swift
/// 请首先加载TensorFlow C语言底层函数库
import TensorFlowAPI
import PerfectTensorFlow
public typealias TF = TensorFlow
```

⚠️注意⚠️ 在使用任何具体函数之前，请首先调用`TF.Open()`打开函数库：

``` swift
// 以下操作会在默认位置加载动态链接库：
// /usr/local/lib/libtensorflow.so
try TF.Open()
```

此处请注意您还可以使用特定目录激活函数库，这在希望机器上安装多个版本时（特别是将CPU和GPU函数库都装到同一个机器上）特别有用：

``` swift
// 下列命令将在指定位置加载动态链接库
try TF.Open("/path/to/DLL/of/libtensorflow.so")
```

## Shape

在Perfect TensorFlow函数库中，Shape定义为如下数据结构，用于存储张量的维度信息：

``` swift
  /// Tensor Shape Object
  public struct Shape {
    public var dimensions = [Int64]()
  }//end class  
```

## Tensor

张量对象被定义为具有具体数据类型的多维数组。

请注意多数情况下您不需要手工创建Tensor对象，而是使用下列两个简洁方法创建：Scalar<T>，用于建立无维度常量；而Array<T>用于建立多维数组。

### Scalar<T>

无维度常量可以理解为维度为0或者1的数组——即只有一个元素。创建该张量的方法，是将具体元素变量作为参数直接调用`Scalar<T>(value: T)`方法：

``` swift
// 创建一个32位整型变量为内容的张量
// 注意如果仅声明Int，则默认为64位
let x = try TF.Tensor.Scalar(Int32(100))

// 创建一个32位浮点变量为内容的张量
// 注意如果不声明Float，则张量会被设置为双精度
let y = try TF.Tensor.Scalar(Float(1.1))

// 创建一个字符串类型的张量：
let s = try TF.Tensor.Scalar("你好，TensorFlow! 🇨🇳🇨🇦")
```

如果希望在创建张量之后读取其张量值，则使用`asScalar<T>()` 方法：

``` swift
let x: Int32 = try tensor.asScalar()
```

### Array<T>

数组类的张量需要在构造时提供维度信息；另外请注意在输入矩阵时需要把多为矩阵压平为单维数组：

``` swift
/* 如果想输入下列矩阵作为张量：
| 1 2 |
| 3 4 |
*/
// 首先将矩阵压平成数组：
let m:[Float] = [[1, 2], [3, 4]].flatMap { $0 }

// 然后再伴随维度信息一起生成新的张量：
let tensor = try TF.Tensor.Array(dimenisons: [2,2], value: m)
```

如果希望将张量内容还原为数组，请调用 `asArray<T>()` 方法：

``` swift
let y: [Float] = try tensor.asArray()
```

⚠️**注意**⚠️ 如果张量为字符串数组，那么不要用上面的方法，而是直接调用`strings`属性，注意返回值为`Data`类型的数组：

``` swift
// datastr 实际上为 [Data] 类型
let datastr = tensor.strings

// 将上述结果映射为 [String] 字符串数组
let str = datastr.map { $0.string }
```

### Matrix

自从 Perfect-TensorFlow v1.2.1 版本开始，您可以直接输入矩阵作为张量了！上述例子现在可以改写为：

``` swift
let M = try TF.Tensor.Matrix([[1, 2], [3, 4]])
```

### 访问张量原始数据

如果您希望在程序中访问张量的原始数据，您可以使用`data`属性，或者调用`withDataPointer()`方法通过指针进行访问（虽然有些技巧性难度，但是性能要快很多）——这种情况下`byteCount`属性（张量数据在内存中的大小）和`type`（张量数据类型）会有很大用处：

``` swift
public class Tensor {
	/// 从张量值中获取一个缓冲区块型数据拷贝
	public var data: [Int8]
	
	/// 获得张量缓冲区内数据长度，单位是字节
    public var bytesCount: Int
    
    /// 获取张量值的类型（元素的类型）
    public var `type`: DataType? 
    
    /// 执行不太安全的指针操作来读取张量数据
    public func withDataPointer<R>(body: (UnsafeMutableRawPointer) throws -> R) throws -> R
}
```

### Shape of a Tensor

要获取张量的维度信息，直接使用`dim`属性：

``` swift
public class Tensor {
	public var dim: [Int64]
}
```

## Operation and Its Builder

运算操作是张量计算图中的一个节点，用于表达张量之间的运算方法。

以下步骤用于说明在流程图内构造运算操作的具体流程：

- 第一步，从运算流程图中导出一个 `OperationBuilder` 辅助构造工具对象。
- 设置目标运算操作的名称、类型和设备
- 设置运算操作有关的张量
- 增加输入输出
- 设置其他有关属性
- 调用 `build()` 方法返回期望的运算操作对象。

请参考下列范例：

``` swift
// 首先构造张量，比如 Tensor.Scalar(100)
let tensor = try TF.Tensor ...

let g = try TF.Graph()

// 从流程图导出一个构造运算操作的实例模板
let operation = try g.opBuilder(name: "Const_0", type: `Const`)

	// 设置属性
	.set(attributes: ["value": tensor, "dtype": tensor.type])
	
	// 生成运算操作
	.build()
```

### OperationBuilder

运算操作辅助构造类 OperationBuilder 定义如下：

``` swift
public class OperationBuilder {

	 /// 构造函数；最好使用graph.opBuilder() 代替
    public init(graph: Graph, name: String, `type`: String) throws
    
    /// 为目标运算增加输入
    public func add(input: Output) -> OperationBuilder 

    /// 位目标运算增加一组输入
    public func add(inputs: [Output]) throws -> OperationBuilder 

    /// 追加控制
    public func add(control: Operation) -> OperationBuilder 

    /// 构造运算
    public func build() throws -> Operation 

    /// 设置设备
    public func `set`(device: String) -> OperationBuilder 
    
    /// 设置属性。属性为一个Swift字典，每一个条目代表一个属性名称。
	 /// 有效值包括： Int64、[Int64]、Float、[Float]、 
	 /// Bool、[Bool]、DataType、[DataType]、String、[String]、
	 /// Shape、[Shape]、Tensor、[Tensor]、
	 /// TensorProto、[TensorProto]、Data
    public func `set`(attributes: [String: Any] = [:]) 
```

### Operation

一旦用辅助工具类OperationBuilder成功构造运算操作，该运算操作内的所有属性和值都可以通过下列方法进行访问：

|接口函数/属性|类型|说明|
|---------------|----|-----------|
|fun attribute(forKey: String)|[AttrValue](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/attr_value.proto)|根据属性名称查看当前运算操作的某个属性|
|var NodeDefinition|[NodeDef](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/node_def.proto)|获取节点定义|
|var name|String|获取运算操作名称|
|var type|String|获取运算操作类型|
|var device|String|获取运算操作设备|
|var numberOfInputs|Int|获取运算操作的输入数量|
|var numberOfOutputs|Int|获取运算操作的输出数量|
|func sizeOfInputList(argument: String)|Int|通过参数字符串获取输入列表|
|func sizeOfOutputList(argument: String)|Int|通过参数字符串获取输出列表|
|var controlInputs|[Operation]|获取控制输入
|func asInput(_ index:Int)|Input|将当前运算操作转化为输入|
|func asOutput(_ index:Int)|Output|将当前运算操作转化为输出|


除此之外，运算操作类还包含下列三个静态方法：

|静态方法|说明|
|-------------|-----------|
|func Consumers(output: Output) -> [Input]|获取输出下的所有消费者|
|func TypeOf(input: Input) -> DataType?| 获取输入类型|
|func TypeOf(output: Output) -> DataType?|获取输出类型|

⚠️**注意**⚠️ 更多常用张量运算操作构造方法请参考[运算流程图](#common-operations)


## Output

TensorFlow中的输出实际上是代表运算操作的一个符号链接，您可以将其简单理解为“带有索引编号的运算操作”： `let output = op.asOutput(0)`

## Graph

运算流程图用于代表张量计算的整个过程。创建图对象的方法有以下几种：

``` swift
public class Graph {

	/// 创建空白流程图，比如 `let x = try TF.Graph()`
	public init () throws 
	
	/// 从参考指针转换为一个流程图
	/// 通常在一组流程图操作时会用到
	public init(handle: OpaquePointer) 
}
```

此外，您还可以将一个预先训练好的模型导入到当前流程图中，其中的“definition”（流程图定义）对象是一个[谷歌缓冲协议](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/framework/graph.proto)，可以在该对象和二进制文件之间互相转换：

``` swift
let g = try TF.Graph()
try g.import(definition: def)
```

### Operations in a Graph

调用`operations`属性可以数组方式获得流程图内的所有运算操作；或者您也可以通过搜索名称的方式来获取具体的操作：

``` swift
// 在流程图中检索一个名称为placeholder的操作
let placeholder = try graph.searchOperation(forName: "placeholder")

// 将运算流程图中的所有操作以数组方式导出
let list = graph.operations
```

#### Common Operations

以下是在流程图内构造运算操作的常用方法：


|名称|说明|举例|
|----|-----------|-------|
|const|构造常量|`let x = try graph.const(tensor: t, name: "Const_0")`|
|placeholder|构造一个占位操作|`let feed = try graph.placeholder(name: "feed")`|
|binaryOp|在两个输出之间进行某种二进制操作 | `func binaryOp(_ `type`: String, _ in1: Output, _ in2: Output, name: String = "", index: Int = 0) throws -> Output `|
|scalar|创建一个32位整数张量|`let ten = try graph.scalar(10, name: "ten")`|
|scalar|创建一个浮点类型张量|`let point = try graph.scalar(0.1, name: "point")`|
|constantArray|创建一个类型为常量数组的张量|`let size = try g.constantArray(name: "size", value: [1024,768])`|
|add|输出求和|`let sum = try graph.add(left:lOutput, right:rOutput, name: "sum")`|
|add|输出求和（操作序号自动设置为零）|`let sum = try graph.add(left:lOp, right:rOp, name: "sumOp")`|
|neg|操作求反|`let neg = try graph.neg(op, name: "MyNeg")`|
|matMul|矩阵叉积|`let m = try graph.matMul(l: aOp, r: bOp, name: "m0", transposeA:false, transposeB: true)` 即 A 矩阵与 B 矩阵的转置进行叉积运算|
|div|除法运算|`let z = try graph.div(x: x, y: y, name: "Div0")`|
|sub|减法运算|`let z = try graph.sub(x: x, y: y, name: "Div0")`|
|decodeJpeg|JPEG图像解码|`let decoded = try graph.decodeJpeg(content: input, channels: 3)`|
|resizeBilinear|双线性图像尺寸调制|`let resizes = try g.resizeBilinear(images: images, size: size)`|
|cast|将输出类型进行强制转换|`let cast = try g.cast(value: jpeg, dtype: TF.DataType.dtFloat)`|
|expandDims|扩大维度|`let images = try g.expandDims(input: cast, dim: batch)`|

上述大部分矩阵运算操作可以在[Perfect TensorFlow 机器视觉演示程序](https://github.com/PerfectExamples/Perfect-TensorFlow-Demo-Vision/blob/master/Sources/main.swift#L62-L72) 中找到例子。

⚠️**注意**⚠️ 如果希望从流程图内加载一个预先保存好的会话模型，请调用`graph.load()`方法，详见[Load A Session](#load-a-session-runner)

## Session

会话对象用于代表按照流程图执行一次完整运算的过程。

参考下列范例，执行运算操作的最简单方法是针对`Runner`对象实例执行下列链式编程：

``` swift
let results = try graph.runner()
	.feed("input", tensor: image)
	.fetch("output")
	.run()
```

具体过程如下：第一步，创建一个运算会话句柄：`let r = try graph.runner()`，然后给这个会话提供实际的数据输入：

``` swift
class Runner {
	/// 为会话提供一个输入，输入来自于之前的某个运算输出，以及一个张量：
	public func feed(_ output: Output, tensor: Tensor) -> Runner
	
	/// 为会话提供一个输入，输入来自于之前的某个运算操作，以及一个张量：
	public func feed(_ operation: Operation, index: Int = 0, tensor: Tensor) -> Runner 
	
	/// 为会话提供一个输入，输入来自于之前的某个运算操作的，以及一个张量；
	/// 此时运算可以用其名称表示，即先按名称搜索到运算，然后在提供输入
	public func feed(_ operation: String, index: Int = 0, tensor: Tensor) throws -> Runner
}
```

与提供输入的方法差不多，您还可以要求会话结果生成具体的输出：

``` swift
class Runner {
	/// 将会话结果定向到某个输出
	public func fetch(_ output: Output) -> Runner
	
	/// 将会话结果定向到某个具体操作；
	/// 如果 index 为零，则等同于fetch(output)
	public func fetch(_ operation: Operation, index: Int = 0) -> Runner
	
	/// 将会话结果定向到某个特定名称的操作：
	public func fetch(_ operation: String, index: Int = 0) throws -> Runner
}
```

然后可以为会话增加目标：

``` swift
class Runner {
	/// 追加目标运算操作
	public func addTarget(_ operation: Operation) -> Runner 
	
	/// 按照名称追加目标运算操作
	public func addTarget(_ operation: String) throws -> Runner
}
```

最后一步是调用`run()`方法执行整个流程图，然后生成一个张量数组：

``` swift
class Runner {
	/// 按照预定义的所有输入输出及目标运算执行一次流程图计算，然后生成一组张量
	public func run() throws -> [Tensor]
}
```

考虑到您可能会针对一个运算会话追加多个输入/输出或者目标操作，那么您可以采用下列方式进行链式编程：

``` swift
let r = try graph
	.feed(op1, tensor: t1).feed(op2, tensor: t2)....
	.fetch(out1).fetch(out2).fetch(out3)....
	.addTarget("opA").addTarget("opB").addTarget("opC")...
	.run()
```

### Load A Session Runner

如果希望导入一个预先定义好的会话过程，首先创建一个空白流程图，然后使用`import`方法导入：

``` swift
let g = try TF.Graph()

// 元数据用于读取预定义模型中的签名信息
let metaBuf = try TF.Buffer()

// 加载预定义会话
let runner = try g.load(
	exportDir: "/path/to/saved/model",
	tags: ["tag1", "tag2", ...],
	metaGraphDef: metaBuf)
	
```

这样的话，如果模型里面包括签名，则可以读出来：

``` swift
if let data = metaBuf.data {
	let meta = try TF.MetaGraphDef(serializedData: data)
	let signature_def = meta.signatureDef["某些签名"] 
	let input_name = signature_def.inputs["某些输入名称"]?.name
	let output_name = signature_def.outputs["某些输出名称"]?.name
	...
}
```

准备好之后，就可以调用`runner.run()`，和前面章节使用方法一样。

### 会话过程的设备属性

从 TensorFlow 1.3.0以上版本开始，可以调用 `session.devices`方法获得设备清单属性，返回值是一个元组数组：

``` swift
let dev = try g.runner().session.devices
print(dev)
// 样本输出：
// ["/job:localhost/replica:0/task:0/cpu:0": (type: "CPU", memory: 268435456)]
```