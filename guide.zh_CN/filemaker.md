# FileMaker

FileMaker服务器连接工具采用XML网页发布界面访问FileMaker服务器。该工具允许用户从程序内访问FileMaker数据库，实现对数据记录的增加、更新和删除操作。

## 系统要求

该模块使用libxml2和libcurl作为基本配置，并使用[Perfect-CURL](https://github.com/PerfectlySoft/Perfect-XML">Perfect-XML</a>和<a href="https://github.com/PerfectlySoft/Perfect-CURL)工具包。

### macOS

macOS目前已经包括了该工具所需的必要库函数，因此不需要手工安装。

### Linux

需要通过apt安装libcurl4-openssl-dev和ibxml2-dev开发包

```
sudo apt-get install libcurl4-openssl-dev libxml2-dev
```

## 配置文件

请在您的Perfect项目中的Package.swift文件增加“Perfect-FileMaker”依存关系：

``` swift
.Package(
	url:"https://github.com/PerfectlySoft/Perfect-FileMaker.git",
	majorVersion: 3
	)
```

### 声明和导入

请在您的Swift源程序开头增加以下声明导入对FileMaker的支持：

``` swift
import PerfectFileMaker
```

## 综述

访问FileMaker数据库的主要对象简述如下。

### struct FileMakerServer 数据库服务器结构

这是访问一个FileMaker服务器的主要程序界面。在访问人和一个具体数据库之前，请为这个结构创建一个实例，并提供服务器主机名或IP地址，端口号和所需的用户名密码。如果指定443端口则后续访问将采用HTTPS数据加密

一旦服务器连接成功并实现实例初始化，您就可以进行访问数据库操作了：

* 罗列可用的数据库资源
* 罗列其中任意数据库的视图
* 罗列每一个视图中的所有字段
* 执行查询

在左右情况下都需要提供回调函数，以保证实现异步操作。如果操作完成，回调函数将根据指定闭包进行操作，操作结果要么成功返回数据库响应，要么会抛出异常。对应操作的响应对象会根据具体操作类型延迟注销（defer）。出错情况下被抛出的异常将提供错误代码和对应的错误消息字符串，类型如下：

``` swift
public enum FMPError: Error {
    /// 错误代码和对应的错误信息
    case serverError(Int, String)
}
```

FileMakerServer struct结构的每个成员定义如下：

``` swift
/// FileMaker Server服务器的连接实例
/// 指定主机、端口、用户名和密码以进行初始化操作。
public struct FileMakerServer {
    /// 用主机名、端口、用户名和密码初始化构造函数。
    public init(host: String, port: Int, userName: String, password: String)
    /// 从服务器端返回数据库清单。
    public func databaseNames(completion: @escaping (() throws -> [String]) -> ())
    /// 根据指定数据库返回视图清单。
    public func layoutNames(database: String, completion: @escaping (() throws -> [String]) -> ())
    /// 获得指定数据库视图的详细信息，包括所有字段和显示在视图上的名称。
    public func layoutInfo(database: String,
                           layout: String,
                           completion: @escaping (() throws -> FMPLayoutInfo) -> ())
    /// 执行查询并返回查询结果记录集。
    public func query(_ query: FMPQuery, completion: @escaping (() throws -> FMPResultSet) -> ())
}
```

当罗列数据库或视图名称时，返回响应将以字符串数组的形式体现。当获取视图信息时，响应对象会转化为FMPLayoutInfo对象；而执行查询时返回的对象为FMPResultSet查询结果记录集。

### struct FMPLayoutInfo视图信息结构

FMPLayoutInfo视图信息结构定义如下：

``` swift
/// 代表了特定视图的元信息数据。
public struct FMPLayoutInfo {
    /// 每个字段或者关联集合都会作为一个清单项。
    public let fields: [FMPMetaDataItem]
    /// 通过名称访问每一个字段或者对应关键字的关联集合。
    public let fieldsByName: [String:FMPFieldType]
}
```

该对象包含试图内所有字段的信息内容。包括字段名称、类型、在视图上显示的名称，以及字段到显示名称之间的对应关系。

字段信息可以通过两种不同方式获取。第一种是通过一个FMPMetaDataItems的数组来表示。下面的枚举类型说明了每个视图条目是一个常规字段还是一个在视图上显示的名称或对应关系。第二种方式是通过一个字典来维护完整的字段名称和字段类型。如果字段代表了一个对应关系，则会用标准的FileMaker PortalName::FieldName （即显示名称::字段名称）语法来表示。

以下是FMPMetaDataItem元数据条目、FMPFieldDefinition字段定义和FMPFieldType类型的枚举：

``` swift
/// 代表了任意一个条目是一个独立的字段定义还是一个关联定义（即字段与视图显示名称的对应关系）。
public enum FMPMetaDataItem {
    /// 一个独立的字段
    case fieldDefinition(FMPFieldDefinition)
    /// 一个对应关系集合，说明了从视图显示名称到所包含的字段。
    case relatedSetDefinition(String, [FMPFieldDefinition])
}
```

``` swift
/// 一个FileMaker字段定义，说明字段的名称和类型。
public struct FMPFieldDefinition {
    /// 字段名称
    public let name: String
    /// 字段类型
    public let type: FMPFieldType
}
```

``` swift
/// FileMaker所有可能的字段类型。
public enum FMPFieldType {
    /// 文字字段
    case text
    /// 数字字段
    case number
    /// 容器字段
    case container
    /// 日期字段
    case date
    /// 时间字段
    case time
    /// 时间戳字段
    case timestamp
}
```

### struct FMPResultSet结果记录集结构

一个FMPResultSet结果记录集对象容纳了一个FileMaker查询的返回结果。该结构包括数据库视图的元信息，还包括数据行数，以及每行数据记录的详细信息。

``` swift
/// 该结果记录集由数据库查询创建。
public struct FMPResultSet {
    /// 数据库元信息。
    public let databaseInfo: FMPDatabaseInfo
    /// 数据视图信息。
    public let layoutInfo: FMPLayoutInfo
    /// 查询所返回的记录行数统计。
    public let foundCount: Int
    /// 由查询创建的数据行清单。
    public let records: [FMPRecord]
}
```

除了之前描述的FMPLayoutInfo struct数据视图结构对象之外，返回结果记录集还会包含一些其它有关数据库的信息：

``` swift
/// 数据库元信息。
public struct FMPDatabaseInfo {
    /// 服务器提供的日期格式。
    public let dateFormat: String
    /// 服务器提供的时间格式。
    public let timeFormat: String
    /// 服务器提供的时间戳格式。
    public let timeStampFormat: String
    /// 数据库中所包含的所有数据记录行总数。
    public let recordCount: Int
}
```

返回的所有数据记录是通过访问结果记录集```FMPResultSet.records```属性实现。这个属性是一个FMPRecords数组，每个数组元素对应了一条数据记录（一行记录）。

``` swift
/// 一个单独的记录集。
public struct FMPRecord {
    /// 记录条目的类型枚举。
    public enum RecordItem {
        /// 一个独立字段。
        case field(String, FMPFieldValue)
        /// 一个关联的结婚，内容是关联的记录清单。
        case relatedSet(String, [FMPRecord])
    }
    /// 记录编号。
    public let recordId: Int
    /// 对应每个关键词的数据记录。
    public let elements: [String:RecordItem]
}
```

每个数据记录都保存了通过字段或视图显示名称来访问其实际值```elements```属性。该字典内的值是```FMPRecord.RecordItem```枚举类型。这些对象意味着如果条目是一个常规字段，就可以通过字段名称直接访问取值，或者是一个关联的记录集，则返回内嵌的记录数组。

每一种情况都是通过FMPFieldValue struct结构来代表字段对应的实际值。

``` swift
/// 返回FileMaker字段数据值
public enum FMPFieldValue: CustomStringConvertible {
    /// 文字字段。
    case text(String)
    /// 数字字段。
    case number(Double)
    /// 容器字段。
    case container(String)
    /// 日期字段。
    case date(String)
    /// 时间字段。
    case time(String)
    /// 时间戳字段。
    case timestamp(String)
}
```

## 查询

调用`FileMakerServer.query`函数能够查询检索指定记录，或操作每一个具体的记录。查询操作可以使用FMPQuery struct结构对象完成。该结构在发送到FileMaker服务器时以一个FileMaker XML标准格式化的字符串实现查询。服务器返回的响应也是XML。该XML响应被自动解析并转化为一个FMPResultSet查询结果记录集对象。

由FMPQuery对象生成的查询字符串格式请参考PDF文件（英文版） [《FileMaker® 服务器12版XML定制发布标准》](https://www.filemaker.com/support/product/docs/12/fms/fms12_cwp_xml_en.pdf)。

数据库查询的创建是通过初始化FMPQuery struct结构对象并对其增加选项而成。每一个选项会返回一个新修改的对象。这种方式的选项可以形成构造最终查询对象的一个选项链。构造完成之后就可以调用```FileMakerServer.query```函数并把这个查询结构对象作为参数传递，这样就可以产生最终的查询结果记录集。

FMPQuery详细定义如下：

``` swift
/// 数据库查询操作。
public struct FMPQuery: CustomStringConvertible {
    /// 构造函数；按照数据库名、视图名和数据库操作初始化。
    public init(database: String, layout: String, action: FMPAction)
    /// 设置记录编号并返回调整后的查询。
    public func recordId(_ recordId: Int) -> FMPQuery
    /// 以分组方式增加待查询字段并返回调整后的查询。
    public func queryFields(_ queryFields: [FMPQueryFieldGroup]) -> FMPQuery
    /// 增加待查询字段并返回调整后的查询。
    public func queryFields(_ queryFields: [FMPQueryField]) -> FMPQuery
    /// 增加用于排序的字段并返回调整后的查询。
    public func sortFields(_ sortFields: [FMPSortField]) -> FMPQuery
    /// 增加预先排序脚本并返回调整后的查询。
    public func preSortScripts(_ preSortScripts: [String]) -> FMPQuery
    /// 增加预先检索脚本并返回调整后的查询。
    public func preFindScripts(_ preFindScripts: [String]) -> FMPQuery
    /// 增加查询后检索的脚本并返回调整后的查询。
    public func postFindScripts(_ postFindScripts: [String]) -> FMPQuery
    /// 设置响应视图并返回调整后的查询。
    public func responseLayout(_ responseLayout: String) -> FMPQuery
    /// 增加响应字段并返回调整后的查询。
    public func responseFields(_ responseFields: [String]) -> FMPQuery
    /// 设置每次读取的最大记录数并返回调整后的查询。
    public func maxRecords(_ maxRecords: Int) -> FMPQuery
    /// 设置在检索集合中可以忽略的记录数量并返回调整后的查询。
    public func skipRecords(_ skipRecords: Int) -> FMPQuery
    /// 返回格式化后的查询字符串
    /// 对于调试来说非常方便
    public var queryString: String
}
```

FMPQuery对象首先通过设置数据库名称、视图名称和操作来初始化。可能的操作包括：

``` swift
/// 数据库操作枚举
public enum FMPAction: CustomStringConvertible {
    /// 在当前查询内进行检索。
    case find
    /// 检索所有记录。
    case findAll
    /// 检索并返回一个随机记录。
    case findAny
    /// 在指定查询基础上创建一个新记录数据
    case new
    /// 通过记录编号和对应的字段/值编辑（更新）数据记录。
    case edit
    /// 根据记录编号删除记录。
    case delete
    /// 通过当前记录编号制作记录副本。
    case duplicate
}
```

许多基于FMPQuery的函数都接受字符串或整型值用于自我解释。凡是接受数组为参数的函数被称为“多次函数”，其新增数据值会被追加到现有数据值的集合中去。

当执行```.edit```（编辑）、```.delete```（删除）或```.duplicate```（复制）操作时，必须通过```FMPQuery.recordId(_ recordId: Int)```设置记录编号。当需要```.find```操作时，指定记录编号会返回对应的记录。

在一个数据视图上实现从其它视图检索字段是可行的。调用```FMPQuery.responseLayout(_ responseLayout: String)```来设置响应操作的视图。默认情况下被检索视图就是响应返回的视图。

如果可能，我们建议您从性能方面的考虑触发，在一个查询结果记录集内返回尽量少的字段内容。期望返回的字段名称可以通过```FMPQuery.responseFields(_ responseFields: [String])```函数进行设置。

字段查询和排序的用法罗列如下：

### 排序

在结果记录集内的记录可以通过一个或多个字段进行排序。排序是通过将期望的字段和排序方法以FMPSortField作为参数调用```FMPQuery.sortFields```函数实现。FMPSortField使用FMPSortOrder枚举类型来确定排序顺序，二者共同定义如下：

``` swift
/// 数据排序顺序。
public enum FMPSortOrder: CustomStringConvertible {
    /// 按照升序方式排序（从小到大，从低到高）。
    case ascending
		/// 按照降序方式排序（从大到小，从高到低）。
    case descending
    /// 通过指定字段自定义方式排序。
    case custom
}

/// 排序字段指示器。
public struct FMPSortField {
    /// 用于排序的字段。
    public let name: String
    /// 预期的排序顺序方式。
    public let order: FMPSortOrder
    /// 通过字段名称和排序方式实现初始化的构造函数。
    public init(name: String, order: FMPSortOrder)
    /// 通过字段名称并默认为升序方式FMPSortOrder.ascending初始化的构造函数。
    public init(name: String)
}
```

### 待查询字段

待查询字段可以被加入到FMPQuery对象中用于说明哪些字段在调用 ```.edit```操作时应该被修改；或者在执行```.find```操作时，哪些字段是用于检索。待查询字段保存了字段名以及对应的目标取值。在```.find```操作时，待查询字段还保存了一个字段级别运算符。这些运算符代表了字段到对应数据之间的关系。比如运算符可以是表达在检索中“大于等于”。默认的字段级别运算符是”以该值开始“的通配符（即FileMaker数据库的标准搜索方式）。

每个待查询字段都是用FMPQueryField对象表示。字段级别运算符是通过FMPFieldOp实现，定义如下：

``` swift
/// 字段级别运算符。
public enum FMPFieldOp {
   case equal /// 等于
   case contains /// 包含
   case beginsWith /// 以之开始
   case endsWith /// 以之结尾
   case greaterThan /// 大于
	case greaterThanEqual /// 大于等于
	case lessThan /// 小于
	case lessThanEqual /// 小于等于
}

/// 待查询字段
public struct FMPQueryField {
   /// 字段名称
   public let name: String
   /// 字段取值
   public let value: Any
   /// 检索运算符
   public let op: FMPFieldOp
   /// 通过字段名称，目标取值和运算符进行初始化的构造函数。
   public init(name: String, value: Any, op: FMPFieldOp = .beginsWith)
}
```

当进行一个```.edit```编辑操作时，可以将待查询字段作为数组参数追加到FMPQuery中去，具体调用方法是```FMPQuery.queryFields(_ queryFields: [FMPQueryField])```函数。在```.edit```编辑操作过程中，如果待查询字段包含字段级别运算符，则这些运算符会被忽略。

当执行一个```.find```检索操作时，待查询字段是由逻辑运算符进行分组的。逻辑运算符确定了哪些字段用于整合在一起用于查询。可能的逻辑运算符即或与非：and、or和not。其含义在于，在结果记录集内查询和选择数据记录时：

* and（逻辑与）：完全符合上述待查询字段的所有记录将整合成一组。
* or（逻辑或）：任何匹配待查询字段的记录将整合成一组。
* not（逻辑非）：如果有记录匹配这一组的查询字段，则记录将会从结果集内被忽略。

待查询字段可以分成不同的组增加到一个FMPQuery查询对象。当FileMaker执行检索时，这些组将会被顺序调用。

逻辑运算符和待查询字段组是通过FMPLogicalOp和FMPQueryFieldGroup分别表示：

``` swift
/// 用于待查询字段组合的逻辑运算符枚举
public enum FMPLogicalOp {
    case and, or, not
}

/// 一个待查询字段的组合结构
public struct FMPQueryFieldGroup {
    /// 该组合的逻辑运算符。
    public let op: FMPLogicalOp
    /// 该组合内的待查询字段清单。
    public let fields: [FMPQueryField]
    /// 通过逻辑运算符和待查询字段清单进行初始化构造函数。
    /// 默认的逻辑运算符是逻辑与FMPLogicalOp.and。
    public init(fields: [FMPQueryField], op: FMPLogicalOp = .and)
}
```

待查询字段组合可以通过```FMPQuery.queryFields(_ queryFields: [FMPQueryFieldGroup])```函数追加到查询中去。

## 举例

以下程序展示了FileMaker数据库的基本操作。

### 罗列所有可用的数据库

以下源代码连接到服务器并返回服务器上所有数据库的清单。

``` swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.databaseNames {
    result in
    do {
        // 获取所有数据库名称
        let names = try result()
        for name in names {
            print("得到数据库名称： \(name)")
        }
    } catch FMPError.serverError(let code, let msg) {
        print("服务器出现错误： \(code) \(msg)")
    } catch let e {
        print("调用过程中出现异常：\(e)")
    }
}
```

### 罗列所有可以使用的数据库视图

在特定数据库中罗列所有可以使用的数据库视图。

``` swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.layoutNames(database: "FMServer_Sample") {
    result in
    guard let names = try? result() else {
        return // 出错
    }
    for name in names {
        print("获得数据视图名称： \(name)")
    }
}
```

### 罗列数据视图所有字段

在特定的数据视图上罗列出所有字段名称。

``` swift
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.layoutInfo(database: "FMServer_Sample", layout: "Task Details") {
    result in
    guard let layoutInfo = try? result() else {
        return // 发生错误
    }
    let fieldsByName = layoutInfo.fieldsByName
    for (name, value) in fieldsByName {
        print("字段\(name) = \(value)")
    }
}
```

### 检索所有记录

执行检索所有记录并打印所有字段及其对应数据取值。

``` swift
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .findAll)
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.query(query) {
    result in
    guard let resultSet = try? result() else {
        return // 出错
    }
    let fields = resultSet.layoutInfo.fields
    let records = resultSet.records
    let recordCount = records.count
    for i in 0..<recordCount {
        let rec = records[i]
        for field in fields {
            switch field {
            case .fieldDefinition(let def):
                let fieldName = def.name
                if let fnd = rec.elements[fieldName], case .field(_, let fieldValue) = fnd {
                    print("常规字段： \(fieldName) = \(fieldValue)")
                }
            case .relatedSetDefinition(let name, _):
                guard let fnd = rec.elements[name], case .relatedSet(_, let relatedRecs) = fnd else {
                    continue
                }
                print("Relation: \(name)")
                for relatedRec in relatedRecs {
                    for relatedRow in relatedRec.elements.values {
                        if case .field(let fieldName, let fieldValue) = relatedRow {
                            print("\t关联字段： \(fieldName) = \(fieldValue)")
                        }
                    }
                }
            }
        }
    }
}
```

### 检索所有记录，包含选择忽略和最大限制

如果需要向查询增加忽略和最大限制这两个参数，请参考以下示例：

``` swift
// 忽略两行记录并返回最多两条记录。
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .findAll)
    .skipRecords(2).maxRecords(2)
...
```

### 检索符合待查询字段匹配的数据记录

检索符合字段“Status”（状态）为“In Progress”（进行中）的数据记录

``` swift
let qfields = [FMPQueryFieldGroup(fields: [FMPQueryField(name: "Status", value: "In Progress")])]
let query = FMPQuery(database: "FMServer_Sample", layout: "Task Details", action: .find)
    .queryFields(qfields)
let fms = FileMakerServer(host: testHost, port: testPort, userName: testUserName, password: testPassword)
fms.query(query) {
    result in
    guard let resultSet = try? result() else {
        return // 出错
    }
    let fields = resultSet.layoutInfo.fields
    let records = resultSet.records
    let recordCount = records.count
    for i in 0..<recordCount {
        let rec = records[i]
        for field in fields {
            switch field {
            case .fieldDefinition(let def):
                let fieldName = def.name
                if let fnd = rec.elements[fieldName], case .field(_, let fieldValue) = fnd {
                    print("常规字段： \(fieldName) = \(fieldValue)")
                    if name == "Status", case .text(let tstStr) = fieldValue {
                        print("状态 == \(tstStr)")
                    }
                }
            case .relatedSetDefinition(let name, _):
                guard let fnd = rec.elements[name], case .relatedSet(_, let relatedRecs) = fnd else {
                    continue
                }
                print("Relation: \(name)")
                for relatedRec in relatedRecs {
                    for relatedRow in relatedRec.elements.values {
                        if case .field(let fieldName, let fieldValue) = relatedRow {
                            print("\t关联字段： \(fieldName) = \(fieldValue)")
                        }
                    }
                }
            }
        }
    }
}
```
