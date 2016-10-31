# MongoDB 客户端

MongoClient客户端类用于初始化到MongoDB服务器的连接。

将MongoClient客户端连接到服务器：

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
```

### 关闭连接

一旦连接建立、打开数据库并打开集合，请用`defer`滞后方法关闭连接，注意关闭顺序与建立连接的顺序正好相反——先关闭集合，然后关闭数据库，最后在关闭服务器连接。

``` swift
defer {
    collection.close()
    db.close()
    client.close()
}
```

### 创建数据库引用

调用`getDatabase`函数返回当前服务器连接内指定的MongoDatabase。

``` swift
let db = client.getDatabase( databaseName: <String> )
```

#### 参数说明

* **databaseName:** 字符串类型的数据库名

### 创建集合参考引用

调用`getCollection`方法能够将客户端连接到当前服务器指定数据库下的目标集合。

``` swift
let collection = client.getCollection( databaseName: <String>, collectionName: <String> )
```

#### 参数说明

* **databaseName:** 字符串类型的数据库名称
* **collectionName:** 字符串类型的目标集合名称

### 获得当前Mongo服务器状态

调用`serverStatus`方法返回代表服务器状态的一个对象，

``` swift
let status = client.serverStatus()
```

### 数据库名称列表

调用`databaseNames`方法以字符串数组形式获取当前所有可用的数据库名称列表


``` swift
let dbnames = client.databaseNames()
```
