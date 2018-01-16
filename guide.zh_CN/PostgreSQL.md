# PostgreSQL

该函数库通过封装libpq客户端程序实现Perfect应用程序访问PostgreSQL服务器。

### 系统要求

### macOS

该函数库需要通过[Homebrew](http://brew.sh/) 安装PostgreSQL。

安装Homebrew的方法：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装Postgres服务器的方法：

```
brew install postgres
```

### Linux

请确保已经安装了libpq-dev。

```
sudo apt-get install libpq-dev
```

### 设置

请在Package.swift文件内增加对“Perfect-PostgreSQL”的依存关系

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-PostgreSQL.git",
	majorVersion: 3
	)
```

>请注意一旦修改了Package.swift文件后，请务必重新编译您的Xcode项目

```
swift package generate-xcodeproj
```

### Import声明和导入

如果在您的源程序内要使用PostgreSQL，请在程序开始部分声明导入库文件：

``` swift
import PerfectPostgreSQL
```

### 快速开始

### 连接到数据库

请采用数据库连接字符串作为参数进行连接。对于字符串和URL地址请详见以下PGConnection章节

``` swift
do {
    let p = PGConnection()
    let status = p.connectdb("host=localhost dbname=postgres")
    defer {
        p.close() // 关闭连接
    }
} catch {
    // 错误处理
}
```
用`p.connectdb()`连接数据库后返回的状态值为`.ok`或者`.bad`。这个状态值有助于您快速判断数据库服务器是否可以使用。

### 运行查询

一旦连接数据库后，您就可以对数据库进行数据存取的操作。

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")
let result = p.exec(
    statement: "
        select datname,datdba,encoding,datistemplate
        from pg_database
    ")
```

查询结果记录集（`result`）是以`PGResult`类型返回，详见以下`PGResult`查询结果有关文档。

### 参数绑定

除了在查询语句中直接写字符串值之外，“参数绑定”是另外一种将数据传递给数据库的方法，需要做的只是通过`params`数组把真实值导入。

``` swift

let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")
let result = p.exec(
    statement: "
        insert into test_db (col1, col2, col3)
        values($1, $2, $3)
    ",params: [6, "你好 世界！", "其他值"])

```

您完全可以选择不采用这种绑定方式传递参数，但是如果不采用的话，实际上把负担都转移到SQL语句字符串的验证上。

### 访问结果记录集

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")
let res = p.exec(statement: "
    select datname,datdba,encoding,datistemplate
    from pg_database
    where encoding = $1",
    params: ["6"])
let num = res.numTuples()
for x in 0..<num {
    let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
    let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
    let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
    let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
    print("第一列=\(c1) 第二列=\(c2) 第三列=\(c3) 第四咧=\(c4)")
}
res.clear()
p.close()

```

上面的例子究竟发生了什么


* `res`被赋值为查询结果记录集，其类型为`PGResult`
* `res.numTuples()`是查询结果记录集的行数量
* `PGResult`是可以遍历的，因此代码`for x in 0..<num {}`用于以顺序方式逐行访问每一条记录。
* `getFieldString`、`getFieldInt`和`getFieldBool` 这些方法用于根据结果记录集所在的行和列读取单元格具体内容。


### 管理数据连接：PGConnection

### 打开一个连接

连接参数可以为以下任意一种格式：普通的“关键词＝值”字符串和符合 RFC 3986 标准的URI唯一资源标识符。

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// 或者
let p = PGConnection()
let status = p.connectdb("postgresql://user:password@localhost:5432/dbname")

```

当使用“关键词＝值”字符串连接时，每一个参数都要采用这种格式。如果有参数要写入一个空值，或者值的内容包含空格，则请用单引号引用，比如：keyword = 'a value'。

```
host=localhost port=5432 dbname=mydb connect_timeout=10

```

关于PostgreSQL关键词清单和说明，详见[libpq连接（英文）](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS)

当采用URI方式连接时，通常格式为：

```
postgresql://[user[:password]@][netloc][:port][/dbname][?param1=value1&...]
```
URI的前缀可以是postgresql://或postgres://。每一个URI的组成部件都是可选的，下面的例子显示了常用的URI语法应用：

```
postgresql://
postgresql://localhost
postgresql://localhost:5433
postgresql://localhost/mydb
postgresql://user@localhost
postgresql://user:secret@localhost
postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp
```

### 关闭一个连接

一旦连接建立并打开后，请注意务必在使用结束后关闭。

使用方法是调用`.finish()`函数

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

defer {
    p.finish()
}

```
>注意`.close()`方法也是可以使用的，其含义与`.finish()`完全一致。这么做的目的是与其它的数据库连接器在函数界面上保持一致性。

### 获得连接状态

连接数据库后可以得到其连接状态：

``` swift
let p = PGConnection()
let connection = p.connectdb("host=localhost dbname=postgres")

print("当前数据库连接状态是：\(p.status)")

defer {
    p.finish()
}

```

### 最新错误状态

调用`.errorMessage()`方法可以获得在数据库连接使用中最后一次出现的错误信息

``` swift
p.errorMessage()
```

### 执行SQL语句

使用`.exec`方法可以将原始SQL语句直接传递给PostgreSQL服务器。SQL语句可以使用单纯字符串或者带参数语句完成。

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// 返回四个字段的类型分别为name, oid, integer, boolean
let result = p.exec(
    statement: "
        select datname,datdba,encoding,datistemplate
        from pg_database
    ")

```
SQL语句可以是人和SELECT、INSERT、UPDATE或DELETE，当然也可以是表格创建或者是完整的数据库设置脚本。

除了在查询语句中直接写字符串值之外，“参数绑定”是另外一种将数据传递给数据库的方法，这种方法能够把手工检查数据有效性的工作交给数据库实现。与其把变量值直接写在SQL字符串里，您可以使用`params`数组把变量值以字符串占位符的方式绑定到语句中


``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")


let result = p.exec(
    statement: "
        insert into test_db (col1, col2, col3)
        values($1, $2, $3)
    ",params: [6, "你好", "其它值"])

```

### 查询结果类型：PGResult

PGResult是一个容器类型，所有`.exec`方法执行后都会返回PGResult。

在SELECT语句执行后返回的`PGResult` 响应中，结果集包含的行数、指定行列的单元格可以按照如下方式进行访问：

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// 返回四个字段类型分别为：name, oid, integer, boolean
let res = p.exec(statement: "
    select datname,datdba,encoding,datistemplate
    from pg_database
    where encoding = $1",
    params: ["6"])

let num = res.numTuples()
for x in 0..<num {
    let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
    let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
    let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
    let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
    print("第一列=\(c1) 第二列=\(c2) 第三列=\(c3) 第四咧=\(c4)")
}
res.clear()
p.close()

```

上面的例子究竟发生了什么

* `res`被赋值为查询结果记录集，其类型为`PGResult`
* `res.numTuples()`是查询结果记录集的行数量
* `PGResult`是可以遍历的，因此代码`for x in 0..<num {}`用于以顺序方式逐行访问每一条记录。
* `getFieldString`、`getFieldInt`和`getFieldBool` 这些方法用于根据结果记录集所在的行和列读取单元格具体内容。

### 返回执行语句后的状态

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")


let result = p.exec(
    statement: "
        insert into test_db (col1, col2, col3)
        values($1, $2, $3)
    ",params: [6, "你好！", "其它值"])

print("插入记录后返回状态为：\(result.status())")

```
`.status()`的结果可以是：

* `.emptyQuery`：发给服务器的SQL语句为空字符串
* `.commandOK`：完成了一个不需要返回数据记录的SQL命令
* `.tuplesOK`：完成了SQL查询并返回结果记录集（如SELECT或SHOW）
* `.badResponse`：服务器响应无法解析
* `.nonFatalError`：出现了一个一般性错误（通知或警告）
* `.fatalError`：出现了一个致命错误
* `.singleTuple`：当前查询命令执行后的结果记录集值只包含一条记录。只有单行查询会返回这种结果（比如要求统计某个数量）
* `.unknown`：错误类型未知，需要检查控制台日志以获取错误相关的更多信息

### 获取结果集内记录的行数量

参考之前的例子（“PGResult查询结果”），SELECT语句将`PGResult`转换为`res`结果记录集。在`res`存储的数据记录行数可以通过使用`.numTuples()`方法进行统计：

``` swift
let num = res.numTuples()
```

### 结果记录集内字段数量统计

`.numFields()`返回字段数量。注意是列数不是行数。

### 判断字段的名称和类型

因为字段并不是按照名字访问而是按照索引顺序来访问的，如果查询结果记录集内的字段未知，则需要去判定字段的名称和类型。以下的例子使用`.fieldName(index)`和`.fieldType(index)`进行判断

``` swift
print("第一列字段的名称为：\(res.fieldName(0))")
print("第一列字段的类型为：\(res.fieldType(0))")
```

### 获取行数据

一旦返回结果记录集，就可以用循环遍历所有记录行。`tupleIndex`参数代表了从零开始的行号。

用`.getField*`方法获得具体类型的单元格。

``` swift
let num = res.numTuples()
for x in 0..<num {
    let c1 = res.getFieldString(tupleIndex: x, fieldIndex: 0)
    let c2 = res.getFieldInt(tupleIndex: x, fieldIndex: 1)
    let c3 = res.getFieldInt(tupleIndex: x, fieldIndex: 2)
    let c4 = res.getFieldBool(tupleIndex: x, fieldIndex: 3)
    print("第一列=\(c1) 第二列=\(c2) 第三列=\(c3) 第四列=\(c4)")
}
```
`.getField*`系列函数用于从行列参数指定的单元格返回对应数据类型的值

* `.getFieldString(tupleIndex: Int, fieldIndex: Int)` 返回字符串类型
* `.getFieldInt(tupleIndex: Int, fieldIndex: Int)` 返回整型值
* `.getFieldBool(tupleIndex: Int, fieldIndex: Int)` 返回布尔类型
* `.getFieldInt8(tupleIndex: Int, fieldIndex: Int)` 返回8位整型
* `.getFieldInt16(tupleIndex: Int, fieldIndex: Int)` 返回16位整型
* `.getFieldInt32(tupleIndex: Int, fieldIndex: Int)` 返回32位整型
* `.getFieldInt64(tupleIndex: Int, fieldIndex: Int)` 返回64位整型
* `.getFieldDouble(tupleIndex: Int, fieldIndex: Int)` 返回双精度浮点类型
* `.getFieldFloat(tupleIndex: Int, fieldIndex: Int)` 返回单精度浮点类型
* `.getFieldBlob(tupleIndex: Int, fieldIndex: Int)` 返回一个字节数组

### 测试字段是否包含一个空值

类似于`.getField*`系列方法，`.fieldIsNull`方法也需要一个行（tuple）和一个列索引。返回值为布尔类型真或假。

``` swift
let nullTest = res.fieldIsNull(tupleIndex: <Int>, fieldIndex: <Int>)
```

### ErrorMessage错误信息

如果需要查看数据查询结果是否包含错误信息，请使用`.errorMessage()`方法：

``` swift
let result = p.exec(statement: "SELECT * FROM x")

print("错误信息：\(result.errorMessage())")

```

### 清除结果记录集内的游标

一旦结果记录集处理完毕，所有留在内存中的记录游标是可以被优化的。清除游标将释放内存。

请调用`.clear()`清除游标

``` swift
let p = PGConnection()
let status = p.connectdb("host=localhost dbname=postgres")

// 返回四个字段的类型name, oid, integer, boolean
let result = p.exec(statement: "...")
// 在此处理结果记录集

// 清除结果记录集内的游标：
result.clear()
p.finish()
```
