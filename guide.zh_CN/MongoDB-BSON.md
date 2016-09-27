# BSON 数据转换

BSON是“二进制JSON”的缩写，是一种形如JSON表达式的、以二进制编码的数据序列化文档。如同JSON一样，BSON支持在其文档中的数组和子文档的嵌套。BSON还包含了某些无法用JSON标准表达的数据类型。比如，BSON是兼容日期类型和二进制类型的。

### 创建一个BSON对象

创建BSON对象由好几种方法：

直接创建一个内容为空的BSON对象

```swift
let bsonEmpty = BSON()
```

或者用一个字节数组作为参数来构造一个BSON文档结构。如果这个字节数组的内容是有效的一个BSON文档对象，就可以直接从字节数组的数据转化为文档内容。


```swift
let bsonBytes = BSON(bytes: [UInt8])
```

或者以一个JSON字符串作为参数来创建一个新的BSON文档，同样将JSON数据转化为BSON内容。

```swift
let bsonJSON = BSON(json: String)
```
最后一种方式是由另外一个BSON实例来创建一个新的BSON对象，相当于整个文档内容被完整复制。

```swift
let bsonBSON = BSON(document: BSON)
```

### 关闭BSON对象

关闭、析构并释放当前BSON文档的方法是：

```swift
let bson = BSON()
defer {
    bson.close()
}
```

### 将BSON对象转化为一个JSON字符串

可以调用`asString`方法把BSON文档转化为一个扩展的JSON字符串。

详见[《MongoDB开发手册——扩展JSON》](http://docs.mongodb.org/manual/reference/mongodb-extended-json/)。

```swift
let bson.close = BSON(document: <BSON>)
defer {
    bson.close()
}
print(bson.asString())
```

请调用`asArrayString`方法以字符串数组的形式获取当前BSON对象包裹在最外层数据。

```swift
print(bson.asArrayString())
```

### 将一个BSON对象转化为一个字节数组

调用`asBytes`方法可以将BSON文档转化为一个8位无符号的字节数组`[UInt8]`。

```swift
let bytesArray = bson.asBytes()
```

### 将数据追加到BSON文档

调用`append`方法可以将数据追加到BSON文档。

在调用`append`追加函数过程中，该函数会使用`@discardableResult`属性。该操作的结果可以被返回或者直接忽略。/p>

如果`append`追加操作返回值为：

* **true:** 真值表示内容已经被成功追加
* **false:** 假值表示追加内容后由于突破文档尺寸限额而导致溢出

#### Append函数的其它调用方法

为当前BSON文档追加一个新的字段，类型为`BSON_TYPE_OID`对象标示符。

```swift
bson.append(key: <String>, oid: <bson_oid_t>)
```

为当前BSON文档追加一个新的字段，类型为`BSON_TYPE_DATE_TIME` 日期时间，字段值来自dateTime参数的内容。

```swift
bson.append(key: <String>, dateTime: <Int64>)
```

为当前BSON文档追加一个新的字段，类型为`BSON_TYPE_DATE_TIME` 日期时间，字段值来自time参数的内容。time_t是UNIX使用UTC计算时间的方法，单位是秒。

```swift
bson.append(key: <String>, time: <time_t>)
```

为当前BSON文档追加一个新的字段，类型为`BSON_TYPE_DOUBLE`双精度浮点数，字段值即浮点数值。

```swift
bson.append(key: <String>, double: <Double>)
```

为当前BSON文档追加一个新的字段，类型为`BSON_TYPE_BOOL`布尔类型。

```swift
bson.append(key: <String>, bool: <Bool>)
```

为当前BSON文档追加一个新的字段，类型为UTF-8标准编码的字符串。

```swift
bson.append(key: <String>, string: <String>)
```

为当前BSON文档追加一个新的字段，类型为一个字节数组缓冲区。

```swift
bson.append(key: <String>, bytes: <[UInt8]>)
```

为当前BSON文档追加一个类型为正则表达式 `BSON_TYPE_REGEX`的新字段。 @regex 就是正则表达式字符串，而 @options 将为正则表达式保存选项。

@options合法选项包括

*   'i' 大小写不敏感
*   'm' 允许匹配多个文档
*   'x' 详细说明模式
*   'l' \w和\W命令依赖于本地语言
*   's' 匹配所有数据('.')
*   'u' 使得\w和\W匹配unicode标准

关于如何构造一个BSON正则表达式，详见[bsonspec.org（英文版）](http://bsonspec.org).

* **key:** 字段关键词
* **regex:** 待追加到BSON的正则表达式
* **options:** 正则表达式选项


```swift
bson.append(key: <String>, regex: <String>, options: <String>)
```

### 向BSON文档追加数组

`appendArray`方法可以向BSON文档追加一个完整的数组。BSON数组形如文档，只不过索引值为关键词。比如，数组的第一个元素关键词是"0"，第二个元素的是"1"。

```swift
bson.appendArray(key: <String>, array: <BSON>)
```

同样还可以为一个数组执行内容追加操作，用`appendArrayBegin` 开始追加，然后再调用`appendArrayEnd`完成追加操作。

`appendArrayBegin`方法向BSON对象追加一个新字段，但是操作并未完成。 @child 参数会被初始化便于向字段增加子数组。子数组会占用BSON文档拥有的内存缓冲，因此会导致上一级缓冲因为由额外开销而增长。为了防止太多内存碎片，最好在建立数组用一条malloc语句来分配内存。

子数组参数 @child 的类型是`BSON_TYPE_ARRAY`，因此在这个数组内的所有关键词都应该是"0"、"1"之类。

```swift
bson.appendArrayBegin(key: <String>, child: <BSON>)
```

而`appendArrayEnd`方法结束向BSON追加数组的过程。Child变量应该在调用之后释放注销，不要再使用。

```swift
bson.appendArrayEnd(key: <String>, child: <BSON>)
```

### 合并多个BSON文档

调用`concat`方法来合并`src`参数中的BSON文档。

```swift
bson.concat(src: <BSON>)
```

### 在BSON文档内统计元素数量

用下面的函数统计BSON文档中的字段（元素）数量。

```swift
bson.countKeys()
```


### 检查当前BSON文档是否包含名为 @key 的字段

该函数是大小写敏感的。如果找到了关键词就返回真值`true` ，否则返回假`false`。

```swift
let hasKey = bson.hasField(key: <String>)
```

### 比较两个BSON文档是否完全相等

```swift
guard let bson == bson2 else {
    return false
}
```

### 为排序而比较两个BSON文档

如果lhs排序在rhs之上，将返回真值`true`，否则返回假`false`。

```swift
guard let bson < bson2 else {
    return false
}
```
