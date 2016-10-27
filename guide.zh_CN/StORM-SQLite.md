# SQLiteStORM

### 在工程中引用

请首先在您的 Package.swift 文件中更新依存关系，以确保必要的函数库能够下载到编译环境用于数据库连接：

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 0, minor: 0)
```


## 创建数据库连接

为了创建数据库连接，请自行更新数据源配置：

``` swift
connect = SQLiteConnect("./mydb")
```
一旦连接对象创建，您就可以通过该类对象进行数据操作：

``` swift
let obj = User(connect)
```
上述例子中，“User（用户对象）”通过数据库连接实现了初始化。

在很多情况中，您可能不希望在初始化对象时就连接数据库。这种情况下，您可以先忽略数据库，在初始化对象之后，随时给该对象的 `.connection` 属性赋值，然后就可以连接了：

``` swift
obj.connection = connect
```
