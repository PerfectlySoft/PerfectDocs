# Swift 对象关系管理：“StORM”函数库

StORM 是为 Swift 语言配套的对象管理函数库（ORM），基于[Perfect](https://github.com/PerfectlySoft/Perfect) 软件架构。

该函数库的设计方向瞄准了易学易用、配置灵活，并为程序员提供程序内的数据结构与数据库内的数据结构的一致性。

尽管历史上 ORM 总是备受争议的一种计算机数据架构理念，ORM 的出现使得程序员不必操心数据库具体操作的细节，就可以设计出非常棒的数据库应用程序。

StORM 文档请参考如下：

StORM 有关的数据库（数据源）文档：

* [SQLite3](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-SQLite.md)
* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-PostgreSQL.md)
* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-MySQL.md)


### 在您的项目中使用本函数库

请根据项目需要选择适合您的数据源，并根据数据源的类型确定在您项目中的 Package.swift 中设置必要的依存关系。

比如，如果需要使用PostgreSQL，则请配置为 PostgresStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/Postgres-Storm.git", majorVersion: 0, minor: 0)
```

比如，如果需要使用 MySQL，则请配置为 MySQLStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/MySQL-Storm.git", majorVersion: 0, minor: 0)
```

比如，如果需要使用 SQLite，则请配置为 SQLiteStORM 程序库：

``` swift
.Package(url: "https://github.com/SwiftORM/SQLite-StORM.git", majorVersion: 0, minor: 0)
```

如果您在使用 Xcode，请务必在改变 `Package.swift` 文件之后，需要再次运行 SPM 管理脚本重建 Xcode 项目：

```
swift package generate-xcodeproj
```

## StORM 有关文档：

[设置数据类：](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Setting-up-a-class.md) 如何创建一个数据类并继承所有 StORM 功能。

[数据记录行操作：存、取和删除](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Saving-Retrieving-and-Deleting-Rows.md) 基本数据库操作。

[StORMCursor：控制行记录的游标](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Cursor.md) 管理查询结果记录集并处理记录分页。

[插入数据行：](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Insert.md) 插入数据时的更多细节介绍。

[更新数据记录：](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Update.md) 更新数据时的更多细节介绍。
