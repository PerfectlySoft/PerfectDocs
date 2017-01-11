# MongoDB 集合
一旦服务器连接成功并完成数据库定义，您就可以展开围绕集合的创建、更新和删除操作，并在集合内查询相关文档。

集合可以通过在连接客户端和数据库内“cascading堆叠”定义：

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
let db = client.getDatabase(name: "test")
let collection = db.getCollection(name: "testcollection")
```
或者在打开打开连接后直接赋值产生：

``` swift
let client = try! MongoClient(uri: "mongodb://localhost")
let collection = MongoCollection(
    client: client,
    databaseName: "test",
    collectionName: "testcollection"
    )
```
### 关闭集合
请务必参考下列代码以保证数据连接得到正常关闭：

``` swift
defer {
    collection.close()
    db.close()          // 如果是通过“cascade堆叠方式”创建
    client.close()
}
```

### 集合名称

参考以下函数以字符串类型获得集合名称：

``` swift
collection.name()
```

### 集合重命名

参考以下代码，用newDbName新数据库名、以及newCollectionName新建集合名称这两个方法为集合与数据库改名。上述方法包括一个选项，允许是否在改名后立刻删除现有集合：

``` swift
collection.rename(
    newDbName: <String>,
    newCollectionName: <String>,
    dropExisting: <Bool>
    )
```

#### 参数说明
* **newDbName:** 数据库的新名称
* **newCollectionName:** 集合的新名称
* **dropExisting:** 布尔变量，用于确定如果新建数据库或新集合后，是否立刻删除现有的集合。如果值为真就删除，否则保留当前集合对象。

上述函数返回值是改名操作的结果。

### 删除集合

请使用`.drop`方法从数据库内删除指定集合。执行该命令删除集合时，与集合有关的索引也会被一同删除。

``` swift
collection.drop()
```

返回值是删除操作的状态结果。

### 向一个集合中插入一个文档。

在当前集合中插入一个文档并返回结果状态：

``` swift
collection.insert(_
    document: <BSON>,
    flag: <MongoInsertFlag>
    )
```
#### 参数说明
* **document:** 待插入的BSON文档对象
* **flag:** 可选参数，即`MongoInsertFlag`插入标志，默认为`.None`（不包含任何标志选项）

返回值是插入操作的状态。

#### MongoInsertFlag插入标志选项

MongoInsertFlag插入标志枚举具有以下可选内容：

* **None:** 不需要采取额外操作，这是默认选项
* **ContinueOnError:** 通知MongoDB在操作时忽略错误
* **NoValidate:** 忽略验证过程

### 更新文档
如果需要更新集合内的文档，请用新的BSON对象和选择器去替换原有文档。

``` swift
collection.update(
    update: <BSON>,
    selector: <BSON>,
    flag: <MongoUpdateFlag>
    )
```
#### 参数说明
* **update:** 准备用于替换旧文档的新BSON文档
* **selector:** 与新BSON文档有关的选择标准
* **flag:** 可选项。`MongoUpdateFlag`更新标志，默认为`.None`（不包括任何更新标志选项）

返回值是更新操作之后的结果状态。

#### MongoUpdateFlag更新标志选项

MongoUpdateFlag更新标志枚举具备以下选项：

* **None:** 不需要任何附加操作，这是默认选项
* **Upsert:** 如果选择器找到了对应的文档则更新，否则（即选择器没有找到匹配的文档）直接做插入操作
* **MultiUpdate:** 如果选择器找到了多个符合匹配条件的文档，就全部更新
* **NoValidate:** 忽略验证过程

### 保存

根据文档参数决定更新一个现有文档或者是插入一个新文档。

``` swift
collection.save(document: <BSON>)
```

#### 参数说明
* **document:** 待保存的BSON文档对象

返回值是保存操作的结果状态

* 如果文档不包括一个`_id`字段，则会创建一个新的文档
* 如果确定了一个具体的`_id`字段，`save`操作会执行一个“upsert”操作；如果找到了匹配该`_id`字段的文档，则进行更新，否则直接插入新文档

``` swift
let bson = BSON()
defer {
    bson.close()
}

bson.append(key: "stringKey", string: "String Value")
bson.append(key: "intKey", int: 42)

let result2 = collection.save(document: bson)

```

### 检索

在集合内筛选文档并返回一个指向选中文档的参考游标。

``` swift
collection.find(
    query: <BSON>,
    fields: <BSON>,
    flags: <MongoQueryFlag>,
    skip: <Int>,
    limit: <Int>,
    batchSize: <Int>
    )
```
#### 参数说明
* **query:** *(Optional)* 查询（可选）：指定查询运算符用于筛选文档。如果希望返回在集合内所有文档，就忽略这个参数或者将一个空文档（{}）作为参数传递给函数
* **fields:** *(Optional)* 字段（可选）：指定一个或多个需要用于筛选文档的字段名称。如果希望返回在集合内所有文档，就忽略这个参数
* **flags:** *(Optional)*  标志（可选）：为当前检索设置查询标志
* **skip:** *(Optional)*  忽略（可选）：忽略给定数量的记录
* **limit:** *(Optional)*  限制（可选）：返回记录时数量不得超过指定限额
* **batchSize:** *(Optional)*  批处理大小（可选）：改变自动遍历文档的数量

#### 返回值
检索操作将返回一个符合当前查询条件的文档游标。当find()检索方法“返回文档”时，返回的实际是指向文档的一个参考游标。

### 检索和修改

修改并返回一个单独的文档。默认情况下，所返回的文档不包含更新时的修改内容。如果希望返回在此基础之上的修改稿，则采用**new**选项。

``` swift
collection.findAndModify(
    query: <BSON>,
    sort: <BSON>,
    update: <BSON>,
    fields: <BSON>,
    remove: <Bool>,
    upsert: <Bool>,
    new: <Bool>
    )
```
#### 参数说明
* **query:** *Optional* 查询（可选）：为修改内容设定的选择过滤条件。待查询字段采用与`db.collection.find()`集合检索方法一致的选择器。尽管查询可能匹配多个文档，`findAndModify()`方法只会选择一个文档用于修改
* **sort:** *Optional* 排序（可选）：如果查询选择器过滤并选择了多个匹配文档的情况下，决定哪一个文档用于修改操作。`findAndModify()`方法将只修改按照此参数排序出的第一个文档
* **update:** 更新：必须确定是删除字段还是更新字段，并针对选中的文档执行更新操作。待更新的字段与更新运算符和更新字段是一致的：针对待修改文档确定要修改的实际字段值
* **fields:** *Optional* 字段（可选）：待返回的字段子集。如果需要把字段返回，则通过这种方式标明`fields: { : 1, : 1, … }`，凡是被标记为1的字段均作为结果的一部分返回
* **remove:** 删除：必须决定是删除字段还是更新字段。请用带查询字段中确定要删除哪些文档。如果这个参数设置为真值则一旦选中文档，被选中文档就会被删除。默认是假，即更新字段而不是删除字段。
* **upsert:** *Optional* 更新并插入（可选）用于在更新字段时的合并操作。如果设置为真值，则在`findAndModify()`方法没有找到任何文档时直接创建一个新文档；或者如果检索到了匹配文档，就用`findAndModify()`执行一个数据更新。为了避免多重反复更新，请确认带查询字段是唯一索引的。默认情况下该参数为假
* **new:** *Optional* 新建（可选）：如果该参数设置为真，则在修改操作完成后返回新修改好的文档，而不是返回原文档。调用`findAndModify()`方法时会为删除操作忽略这个新建选项。默认情况下这个参数为假。

### 统计数量

返回`find()` 。该`count()`计数方法不会执行`find()`操作，而是返回匹配查询条件的结果数量。

``` swift
collection.count(
    query: <BSON>,
    fields: <BSON>,
    flags: <MongoQueryFlag>,
    skip: <Int>,
    limit: <Int>,
    batchSize: <Int>
    )
```

#### 参数说明
* **query:** 查询检索条件
* **fields:** 字段（可选）：确定用于匹配文档的字段名称。如果希望返回匹配文档的所有字段，就忽略这个参数
* **flags:** 标志（可选）：用于设置当前检索的查询标志
* **skip:** 忽略（可选）：确定需要忽略的记录数量
* **limit:** 限额（可选）：要求检索返回时记录总数不能超过限额数量
* **batchSize:** 批处理尺寸（可选）：改变自动遍历文档用的数量

#### 返回值
将返回符合`find()`检索匹配条件的记录总数。`count()`方法不会执行`find()`检索操作，而只是返回查询匹配出的文档数量

### 删除
删除由选择过滤器筛选出的文档：

``` swift
collection.remove(
    selector: <BSON>,
    flag: <MongoRemoveFlag>
    )
```

#### 参数说明
* **selector:** 符合筛选条件的BSON文档
* **flag:** *Optional* 标志（可选）：用于删除操作的MongoRemoveFlag删除操作，默认是.None（不需要任何选项标志）

### 创建索引
在集合上创建索引。

``` swift
collection.createIndex(
    keys: <BSON>,
    options: <MongoIndexOptions>
    )
```

#### 参数说明
* **keys:** 形如字段/值对的文档，其字段名称就是索引字段，而字段值代表了该字段索引的类型。如果需要按照字段进行升序索引，则将值设置为1；对于降序索引，请将字段值设置为－1
* **options:** *Optional* 选项（可选）：该参数也是一个文档，用于控制索引的创建。详见MongoIndexOptions索引选项

### 删除索引
从集合中删除一个指定的索引。

``` swift
collection.dropIndex(name: <String>)
```

### 返回统计
返回根据选项文档格式化的统计信息。

``` swift
collection.stats(options: <BSON>)
```

该选项参数可以包括以下字段和值：

* **scale:** *number, Optional* 数量单位（数值类型，可选）：该参数用于显示条目的数量单位，默认输出结果以字节计算。如果要以千字节代替字节作为数量单位，请将scale的值设置为1024

* **indexDetails:** *boolean, Optional* 索引详情（布尔类型，可选）：如果设置为真，则统计函数stats()会在集合统计的基础上返回索引的详细内容。这个可选项只有WiredTiger存储引擎上可以使用，默认为假，也就是不返回索引详情

* **indexDetailsKey:** *document, Optional* 索引键详情（文档类型，可选）：如果indexDetails被设置为真值，您可以使用indexDetailsKey索引键相应来过滤指定键的索引详情。只有与indexDetailsKey精确匹配的索引才会被返回。如果没有找到匹配项，索引细节将显示所有索引的统计数据

* **indexDetailsName:** *string, Optional* 索引细节名称（字符串，可选）：如果indexDetails被设置为真值，您可以使用indexDetailsName来设置要显示细节的索引名称。只有与indexDetailsName名称完全精确匹配的索引才会被返回。如果没有找到匹配项，索引细节将显示所有索引的统计数据

### 验证

验证一个集合。该方法扫描并校验一个集合的数据结构，然后返回一个单独的文档用于描述逻辑集合和数据实际展示之间的逻辑关系。

``` swift
collection.validate(full: <bool>)
```

参数“`full`全面校验”是可选的。如果设置为真则进行全面验证并返回完整的统计数据。MongoDB默认情况下会关闭全面校验选项，因为这个校验过程可能会大量消耗系统资源。

### GetLastError获取最近发生的错误

`getLastError()`方法返回一个BSON文档用于描述最后一次交易结果

``` swift
collection.getLastError()
```
