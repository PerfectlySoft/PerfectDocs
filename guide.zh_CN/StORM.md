# Swift 对象关系管理：“StORM”函数库

StORM 是为 Swift 语言配套的对象管理函数库（ORM），基于[Perfect](https://github.com/PerfectlySoft/Perfect) 软件架构。

该函数库的设计方向瞄准了易学易用、配置灵活，并为程序员提供程序内的数据结构与数据库内的数据结构的一致性。

## StORM 文档内容
[类对象设置](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Setting-up-a-class.md) 如何从StORM创建一个类对象并实现绑定数据表格。

[数据记录增删改](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Saving-Retrieving-and-Deleting-Rows.md) 基本数据库操作

[数据记录游标](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Cursor.md) 管理查询结果分页。

[插入数据行](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Insert.md) 关于插入数据行的更多细节。

[数据行更新](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Update.md) 关于数据行修改的更多细节。

## StORM 有关的数据库（数据源）文档：

* [SQLite3](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-SQLite.md)
* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-PostgreSQL.md)
* [MySQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-MySQL.md)
* * [Apache CouchDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-CouchDB.md)


### 在您的项目中使用本函数库

请根据项目需要选择适合您的数据源，并根据数据源的类型确定在您项目中的 Package.swift 中设置必要的依存关系。

比如，如果需要使用PostgreSQL，则请配置为 PostgresStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/Postgres-StORM.git", majorVersion: 3)
```

比如，如果需要使用 MySQL，则请配置为 MySQLStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-StORM.git", majorVersion: 3)
```

比如，如果需要使用 SQLite，则请配置为 SQLiteStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 3)
```

比如，如果要使用 CouchDB，则请配置为 CouchDBStORM 程序库:

``` swift
.Package(url: "https://github.com/SwiftORM/CouchDB-StORM.git", majorVersion: 3)
```

如果您在使用 Xcode，请务必在改变 `Package.swift` 文件之后，需要再次运行 SPM 管理脚本重建 Xcode 项目：

```
swift package generate-xcodeproj
```
