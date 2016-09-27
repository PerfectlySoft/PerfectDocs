# MongoDB 数据库

MongoDB Database 类用于根据MongoClient客户端实例来访问服务器上的命名数据库。

创建一个新的Mongo Database连接：

```swift
let database = try! MongoDatabase(
    client: <MongoClient>,
    databaseName: <String>
    )
```

### 关闭连接

一旦连接建立、打开数据库并打开集合，请用`defer`滞后方法关闭连接，注意关闭顺序与建立连接的顺序正好相反——先关闭集合，然后关闭数据库，最后在关闭服务器连接。

```swift
defer {
    collection.close()
    db.close()
    client.close()
}
```

### 删除数据库

下面的函数可以删除当前数据库并删除所有相关的数据文件。

```swift
database.drop()
```

### 获得当前数据名

调用`name()`函数返回当前数据库名称。

```swift
let name = database.name()
```

### 创建一个新的集合

```swift
database.createCollection(name: <String>, options: <BSON>)
```

#### 参数说明

* **name:** 用于代表新建集合的字符串名称
* **options:** 新建集合的选项，用BSON文档类型表示

### 通过名称创建MongoDB的集合引用

调用`getCollection`以创建对一个MongoCollection集合对象的引用：


```swift
let collection = database.getCollection(name: <String>)
```

### 当前数据库集合清单

调用`collectionNames`可以字符串数组的形式获得当前数据库内的集合列表：


```swift
let collection = database.collectionNames()
```
