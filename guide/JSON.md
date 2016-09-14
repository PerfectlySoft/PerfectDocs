## JSON数据转换

Perfect通过一系列Swift自建数据类型的扩展实现了基本的JSON编码和解码工具。解码是通过在Swift字符串类型基础上实现的扩展。

请注意虽然Perfect的JSON工具功能强大，但对您的系统而言不是必须的，请根据需要自行选择引用该工具库内的函数。

如果需要使用本系统，请首先在您的源代码开始部分确保PerfectLib库函数已经声明导入：

```swift
import PerfectLib
```

### 将数据编码为JSON格式

您可以将以下数据类型直接转换为JSON字符串：

* String 字符串
* Int 整型
* UInt 无符号整型
* Double 双精度浮点型
* Bool 布尔型
* Array<Any> 任意类型的数组
* Dictionary<String, Any> 以字符串为关键词的字典
* Optional 可选类型
* 从JSONConvertibleObject对象继承而来的定制类型

⚠️注意⚠️对于可选类型而言，只有包含上述任意一种类型的Optional类型才是可以直接转换的。对于值为`nil`的Optionals类型来说，JSON字符串输出结果将会是"null"。

为了实现上述变量类型的编码，请调用上述对象的```jsonEncodedString()```函数。这个函数是Perfect专门做的扩展。该函数有可能会抛出```JSONConversionError.notConvertible```无法转换的异常。

举例

```swift
let scoreArray: [String:Any] = ["第一名": 300, "第二名": 230.45, "第三名": 150]
let encoded = try scoreArray.jsonEncodedString()
```

编码结果是如下字符串：

```
{"第二名":230.45,"第一名":300,"第三名":150}
```

### 解码JSON数据

包含JSON格式数据的字符串可以用```jsonDecode()```函数解码。如果格式有问题，该函数会抛出```JSONConversionError.syntaxError```语法错误异常。

```swift
let encoded = "{\"第二名\":230.45,\"第一名\":300,\"第三名\":150}"
let decoded = try encoded.jsonDecode() as? [String:Any]
```

对上述字符串的解码将会生成下列内容的字典类型：

```
["第二名": 230.44999999999999, "第一名": 300, "第三名": 150]
```

由于解码JSON字符串可能产生任意数据值，因此最常见的方法是用JSON对象（字典）或者数组进行处理。您需要根据结果自行按照预期类型进行转换。

#### 解码后的数据使用

因为解码后的结果总是`[String:Any]`字典或者`[Any]`数组，因此您需要其包含的数据转换为预期类型，比如：

```swift
var firstPlace = 0
var secondPlace = 0.0
var thirdPlace = 0

let encoded = "{\"第二名\":230.45,\"第一名\":300,\"第三名\":150}"
guard let decoded = try encoded.jsonDecode() as? [String:Any] else {
    return
}

for (key, value) in decoded {
    switch key {
    case "第一名":
        firstPlace = value as! Int
    case "第二名":
        secondPlace = value as! Double
    case "第三名":
        thirdPlace = value as! Int
    default:
        break
    }
}

print("前三名：\r" + "第一名" + "\(firstPlace)" + " 分\r" + "第二名：" + "\(secondPlace)" + " 分\r" + "第三名：" + "\(thirdPlace)" + " 分")
```

输出结果为：

```
前三名：
第一名：300分
第二名：230.45分
第三名：150分
```

#### 从JSON数据中解码空值

由于JSON的空值是没有类型的，系统会将空值替换为一个```JSONConvertibleNull```对象。比如：

```swift
let jsonString = "{\"第一名\":300,\"第四名\":null,\"第二名\":230.45,\"第三名\":150}"

if let decoded = try jsonString.jsonDecode() as? [String:Any] {
    for (key, value) in decoded {
        if let value as? JSONConvertibleNull {
            print("字段\"\(key)\"为空值")
        }
    }
}
```

输出为：

```
字段"第四名"为空值
```

### 可转换为JSON的对象

Perfect的JSON转换工具库提供为定制类的编码解码功能。只要从JSONConvertibleObject基类继承即可，如下示例：

```swift
/// 从基类继承为一个可以转化为JSON格式的定制对象。
public class JSONConvertibleObject: JSONConvertible {
    /// 默认构造函数
    public init() {}
    /// 获得JSON键／值
    public func setJSONValues(_ values:[String:Any]) {}
    /// 根据JSON键／值设置对象属性。
    public func getJSONValues() -> [String:Any] { return [String:Any]() }
    /// 将对象编码为JSON文本
    public func jsonEncodedString() throws -> String {
        return try self.getJSONValues().jsonEncodedString()
    }
}
```

任何需要使用JSON编解码的对象都首先要将该对象注册到系统中去。注册工作需要在您的应用程序启动时完成。调用```JSONDecoding.registerJSONDecodable```函数完成对象注册。该函数定义如下：

```swift
public class JSONDecoding {
    /// 该函数为基于JSON成员数据定制对象返回一个新的实例。
    public typealias JSONConvertibleObjectCreator = () -> JSONConvertibleObject
    static public func registerJSONDecodable(name: String, creator: JSONConvertibleObjectCreator)
}
```

注册对象是需要一个唯一的命名。同样还需要一个`creator`函数用于在需要时创建一个新的对象实例。

当系统对一个```JSONConvertibleObject```对象编码时，会调用对象的```getJSONValues```函数。该函数会返回一个`[String:Any]`字典，该字典包含了用于给这个对象编码的所有的字段和属性值。这个字典**必须**要包含一个声明其对象类型的字段。而这个类型字段的值也 **必须** 是与该对象在程序开始阶段注册的名称一致的名字。对应该属性值的字段由```JSONDecoding.objectIdentifierKey```属性而定。

当系统解码这样一个对象时，系统会首先寻找```JSONDecoding.objectIdentifierKey```值，然后在查找之前注册的对象`creator`构造函数。随后系统会根据这个类型和构造函数自动创建一个新对象并调用```setJSONValues(_ values:[String:Any])``` 函数设置各字段值。调用该函数会用一个包含所有解码数据的字典作为参数传递过去。这些属性值会与之前由```getJSONValues```编码函数返回的内容进行匹配。在```setJSONValues```函数中，对象会恢复所有属性与数据。

下面的例子演示了如何定义一个定制的```JSONConvertibleObject```对象，以及如何将其转换为一个JSON字符串。然后再进行解码并与原对象进行比较。⚠️注意⚠️在本例子中对象通过调用```getJSONValue```函数直接把一个命名字段的属性值从字典中抽取出来，而且允许在字典内不包含指定字段的情况下返回一个默认值。

该例子分成了以下几个部分逐一说明。

类定义

```swift
class User: JSONConvertibleObject {
    static let registerName = "user"
    var firstName = ""
    var lastName = ""
    var age = 0
    override func setJSONValues(_ values: [String : Any]) {
        self.firstName = getJSONValue(named: "firstName", from: values, defaultValue: "")
        self.lastName = getJSONValue(named: "lastName", from: values, defaultValue: "")
        self.age = getJSONValue(named: "age", from: values, defaultValue: 0)
    }
    override func getJSONValues() -> [String : Any] {
        return [
            JSONDecoding.objectIdentifierKey:User.registerName,
            "firstName":firstName,
            "lastName":lastName,
            "age":age
        ]
    }
}
```
注册定义好的类信息

```swift
// 运行一次即可
JSONDecoding.registerJSONDecodable(name: User.registerName, creator: { return User() })
```
对象编码：

```swift
let user = User()
user.firstName = "Donnie"
user.lastName = "Darko"
user.age = 17

let encoded = try user.jsonEncodedString()
```
编码后的数据看起来像这样：

```
{"lastName":"Darko","age":17,"_jsonobjid":"user","firstName":"Donnie"}
```
对象解码：

```swift
guard let user2 = try encoded.jsonDecode() as? User else {
    return // 出错
}

// 验证属性值是否一致
XCTAssert(user.firstName == user2.firstName)
XCTAssert(user.lastName == user2.lastName)
XCTAssert(user.age == user2.age)
```

### JSON转换错误

在JSON编码解码过程中，系统可能会抛出一个```JSONConversionError```转换异常，定义如下：

```swift
/// 在JSON编解码过程中可能发生的错误异常。
public enum JSONConversionError: ErrorProtocol {
    /// 对象不支持JSON转换。
    case notConvertible(Any)
    /// 提供的字段不是字符串。
    case invalidKey(Any)
    /// JSON文本内由语法错误。
    case syntaxError
}
```
