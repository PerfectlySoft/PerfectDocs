# Apache CouchDB

CouchDB 数据库连接器提供了 Swift 程序到 Apache CouchDB 服务器的访问，实现文档的增删改和搜索功能


## 系统要求

该模块使用 Perfect-CURL 函数库以及libcurl

### macOS

macOS 目前版本支持该模块，无需追加其他依存关系

### Linux
请安装 libcurl4-openssl-dev 程序库

```
sudo apt-get install libcurl4-openssl-dev
```

## 设置
请在您的 Package.swift 文件中增加 “Perfect-CouchDB” 依存关系：

``` swift
.Package(
    url:"https://github.com/PerfectlySoft/Perfect-CouchDB.git",majorVersion: 3)
```

### 头文件设置

任何需要使用本函数库的源程序开头，请引用：

``` swift
import PerfectCouchDB
```

## 概述

以下是该项目模块的详细介绍：

### CouchDB()

这是连接到 CouchDB 服务器的对象类，包括以下四个公开属性：

* debug: 用于决定是否在终端控制台和日志内输出调试信息。默认为 false；
* database: 该实例的当前数据库；
* connector: 当前实例的连接信息；
* authentication: 身份验证的设置内容；

#### Server API Functions

* serverInfo() - 返回当前服务器信息
* serverActiveTasks() - 返回当前服务器上活动的任务清单
* listDatabases() - 返回服务器上所有活动的数据库

#### 数据库操作

用`databaseExists(String)` 方法来检查以参数值命名的数据库是否存在，如果存在则返回true（真），否则返回false（假）。

``` swift
db.databaseExists("mydb")
```

删除数据库：`databaseDelete(String)`，其参数为待删除数据库的名字。执行后将返回一个 `CouchDBResponse`对象，内容是操作结果。

``` swift
db.databaseDelete("mydb")
```

数据信息可以通过 `databaseInfo(String)`方法完成，返回值是一个`CouchDBResponse, [String:Any]`的tuple元组，其中 CouchDBResponse 是HTTP请求的响应结果，比如(200 OK; 404 Not Found)之类，而[String:Any]则是响应结果变量/名称构成的字典。

``` swift
db.databaseInfo("mydb")
```

响应结果信息包括：

* committed\_update_seq (Int) – 提交更新的数量
* compact_running (Bool) – 如果数据库上在运行压缩程序的话，这个值会为真。
* db_name (String) – 数据库名称
* disk\_format_version (Int) – 数据所在的物理磁盘版本。
* data_size (Int) – 在数据库文件内实际的数据所占字节数量。
* disk_size (Int) – 数据库文件实际尺寸，注意视图和索引不包括在这个尺寸计算中。
* doc_count (Int) – 具体数据库内的文档数量。
* doc\_del_count (Int) – 已删除的文件数量。
* instance\_start_time (String) – 数据库打开的时间戳，自unix epock纪元开始的毫秒数。
* purge_seq (Int) – 执行垃圾清空所用操作的数量。
* update_seq (Int) – 当前数据库更新操作的数量

#### 增加一个文档

增加文档，请采用以JSON结构为参数的`addDoc`方法。

如果 JSON 结构包括了 \_id 字段，则文档会按照指定的ID编号进行创建。如果没有包括 \_id 字段，则系统会根据实际 UUID 算法自动创建一个ID。

参数：

* db: 数据库名称
* doc: 希望保存的JSON文档

待存文档可以是一个 JSON 编码字符串，或者一个 [String: Any] 字典类型。

返回值是一个 CouchDBResponse 和 [String:Any]字典组成的tuple元组。

``` swift
let data = ["one":"ONE","two":"TWO"]
db.addDoc("mydb", doc: data)
```

创建文档的另一个方法是 `create()` 方法。这种方法能够创建一个新文档，或者按照现有文档创建一个新修订版本。与 `addDoc` 方法不同，在此必须为请求指定一个文档 ID 编号。

参数：

* db – 数据名称
* docid – 文档 ID 编号
* doc - [String: Any] 用于存储数据的 JSON 字典。

响应 JSON 对象：

* id (string) – 文档 ID 编号
* ok (boolean) – 操作状态
* rev (string) – 修订版本 MVCC 令牌

状态代码：

* 201 Created – 文档创建成功，并存储在磁盘上
* 202 Accepted – 文档数据已接收，但尚未写入磁盘
* 400 Bad Request – 调用时发生参数或请求数据体错误
* 401 Unauthorized – 没有足够的权限完成该项操作
* 404 Not Found – 指定的数据库或者文档编号不存在
* 409 Conflict – 指定文档的ID编号或版本修订代码已经存在并发生冲突

``` swift
db.create("mydb", docid: String, doc: [String: Any])
```

#### 复制文档

`copy` 方法能够为指定文档创建拷贝。参数包括原文件ID编码和目标文档等参数。

参数：

* db – 数据库名称
* docid – 文档 ID 编号
* rev (string) – 可选项，文档版本修订号
* destination – 目标文件

响应结果JSON对象：

* id (string) – 文档 ID 编号
* ok (boolean) – 操作状态码
* rev (string) – 修订版本 MVCC 令牌

状态代码：

* 201 Created – 文档创建成功，并存储在磁盘上
* 202 Accepted – 文档数据已接收，但尚未写入磁盘
* 400 Bad Request – 调用时发生参数或请求数据体错误
* 401 Unauthorized – 没有足够的权限完成该项操作
* 404 Not Found – 指定的数据库或者文档编号不存在
* 409 Conflict – 指定文档的ID编号或版本修订代码已经存在并发生冲突

``` swift
db.copy("mydb", docid: String, doc: [String:Any], rev: String, destination: String)
```

#### 搜索文档

`filter`过滤器方法能够在指定的数据库中返回所有期望的文档。返回结果是一个JSON数据结构，包括了所有文档清单和基本内容，以及各个文档的ID编码、版本号和从文档\_id产生的钥匙码。

参数：

* db – 数据库名称
* queryParams - CouchDBQuery对象类型的查询参数

响应结果JSON对象：

* offset (Int) – 文档清单的开始偏移量
* rows (Array) – 行视图对象数组。默认返回值只有文档ID编号和修订版本编号。
* total_rows (Int) – 当前视图内的数据文档数量。请注意这个数量不是在实际查询中返回的行数量。
* update_seq (Int) – 获取当前数据库的更新序列

``` swift
let query = CouchDBQuery()
// 请参阅 CouchDBQuery 章节中说明的参数配置方法。
db.filter("mydb", query)
```

形成查询的另一种方法是采用`find` 方法。二者区别在于函数接口的可选项。`find`采用的是一个声明型JSON查询语法。

CouchDBQuery 参数支持：

* selector ([String:Any]) – 用于过滤选择文档标准的 JSON 对象，详情请见选择器selector语法。
* limit (Int) – 返回结果最大限额。默认是25.可选项。
* skip (Int) – 忽略若干个检索记录。可选项。
* sort ([String:Any]) – 排序顺序的JSON字段数组。可选项。
* fields ([String:Any]) – 用于描述返回结果中必须包括的字段内容。如果忽略该参数，则返回整个对象的所有字段。详情请参考字段过滤有关章节。可选项
* use\_index ([String:Any]) – 指定索引构造查询

响应结果JSON对象：

* docs (object) – 符合条件的文档对象

状态代码：

* 200 OK – 查询已完成
* 400 Bad Request – 查询内容无效
* 401 Unauthorized – 没有足够的权限读取数据
* 500 Internal Server Error – 服务器内部出现错误

``` swift
let findObject = CouchDBQuery()
findObject.selector = data
findObject.limit = 10
findObject.skip = 0

do {
	let (code, response) = try db.find("mydb",queryParams: findObject)
} catch {
	print("\(error)")
}
```

#### 获取指定文档

如果希望从数据库中获取特定文档，请使用 get() 方法。

该方法从指定的数据库中根据文档id编号获取数据；如果不特别指定版本修订号，则返回该文档的最新版本。

参数：

* db – 数据库名称
* docid – 文档 ID 编号

查询参数：

* attachments (Bool) – 是否在响应中包括附件数据体。默认为false。
*  att\_encoding_info (Bool) – 如果特定附件是压缩的，是否包括编码信息。默认为false。
* atts_since (array) – 只包含自指定版本之后的数据附件，但不包括指定版本的数据附件，可选。
* conflicts (Bool) – 是否包括文档中的冲突信息。默认为false。
* deleted_conflicts (Bool) – 是否包括已删除的冲突版本。默认为false。
* latest (Bool) – 强制获取最新的“分支（leaf）”版本，无论请求的是哪一个版本。默认为false。
* local_seq (Bool) – 包括最新的文档更新序列。默认为false。
* meta (Bool) – 和确认是否指定所有冲突版本、已删除\_conflicts和open_revs查询参数。默认为false。
* open_revs (array) – 获取指定分支版本的文档。此外，还可以选择所有分支版本。可选项。
* rev (String) – 返回指定版本的文档。可选项
* revs (Bool) – 返回所有已知文档的版本清单。默认为false
* revs_info (Bool) – 包含所有已知版本的详细信息。默认为false

注意所有除docid之外的查询参数都是可选的。
	
响应结果JSON对象：
	
* _id (string) – 文档 ID 编号
* _rev (string) – 修订版本 MVCC 令牌
* _deleted (boolean) – 删除标志。如果文档被删了，就会出现这个标志
* _attachments (object) – 如果文档有附件的话，这里会放置附件。
* _conflicts (array) – 版本冲突清单。如果查询时设置了参数`conflicts=true`，则如果有冲突就会在这里显示出来。
* \_deleted\_conflicts (array) – 已删除版本的清单。如果参数中设置了`deleted_conflicts=true`，则一旦有删除的版本，就会在这里列出。
* \_local\_seq (string) – 当前数据库的文档更新序列。如果查询时包括了`local_seq=true`后，这里会列出相关信息。
* \_revs\_info (array) – 本地版本的对象信息及状态。如果设置了`open_revs`则会显示。
* _revisions (object) – 本地版本清单。如果查询时设置了`revs=true`则会显示。

状态编码：

* 200 OK – 查询完成。
* 304 Not Modified – 文档子指定版本以来尚未修改。
* 400 Bad Request – 查询或版本的格式无效。
* 401 Unauthorized – 没有读取权限
* 404 Not Found – 文档未找到

``` swift
db.get(
		"mydb",
		docid: "docidString",
		attachments: false,
		att_encoding_info: false,
		atts_since: [String](),
		conflicts: false,
		deleted_conflicts: false,
		latest: false,
		local_seq: false,
		meta: false,
		open_revs: [String](),
		rev: "specificRevision",
		revs: false,
		revs_info: false
)
```

#### 更新文档

如果需要更新现有文档，则必须在rev参数中设置当前版本号。

参数：

* db – 数据库名称
* docid – 文档 ID 编号
* doc – [String: Any] 字典格式的文档数据
* rev – 文档修订版本

返回JSON对象：

* id (string) – 文档ID编号
* ok (boolean) – 操作状态
* rev (string) – 版本修订 MVCC 令牌

``` swift 
db.update("mydb", docid: String, doc: [String:Any], rev: String)
```

#### 删除文档

`delete`方法将特定文档标记为删除，即将文档_deleted字段设置为真。设置后将无法被查询到，但是仍然存储在数据库中。使用时必须使用rev参数设置当前（最新）版本。

> 注意：CouchDB 不会彻底删除指定文档，而是会留下一个文档的基本信息，称之为“tombstone”（墓碑），设置的目的是为了实现数据库集群同步复制。

参数：

* db – 数据库名
* docid – 文档ID编号
* rev - 版本修订信息

响应的JSON结果返回：

* id (string) – 文档ID编号
* ok (boolean) – 操作结果状态
* rev (string) – 版本修订 MVCC 令牌

状态代码：

* 200 OK – 文档已删除
* 202 Accepted – 请求已接收，但尚未写入磁盘
* 400 Bad Request – 请求内容数据体或参数无效
* 401 Unauthorized – 没有足够的权限写入
* 404 Not Found – 指定的数据库或者文档编号不存在
* 409 Conflict – 冲突：目标版本不是最新版本。

``` swift
db.delete("mydb", docid: String, rev: String)
```

### CouchDBAuthentication() 认证信息

该结构是用于保存 CouchDB 验证信息。

* **authType**: 认证模式，包括 .none（无身份验证）、.basic（基本验证）或.session（会话）
* **username**: 用户名
* **password**: 密码
* **token**: 如果认证类型为.session，则是会话的令牌信息。如果认证类型不是.session则该属性无效。

用户登录可以通过以下样例方式实现：

``` swift
let auth = CouchDBAuthentication("username","pwd")
let auth = CouchDBAuthentication("username","pwd", auth: .basic)
```

计算 base64 令牌编码：

``` swift
let auth = CouchDBAuthentication("username","pwd")
let token = auth.basic()
```

### CouchDBConnector()

该数据结构用于保存数据库连接信息。

* **ssl**: Bool - 是否启用SSL加密模式。默认是false，即不加密
* **host**: String - CouchDB 服务器域名或者 IP 地址。默认为localhost
* **port**: Int - CouchDB 服务器，默认端口为 5984

``` swift
var thisConnector = CouchDBConnector()
thisConnector.host = "192.168.0.12"
thisConnector.ssl = true
thisConnector.port = 5984
```

### CouchDBQuery()

CouchDBQuery 是用于控制查询参数实现检索的数据结构，包括以下属性：

* **conflicts** : Bool – 是否在查询结果中包括版本冲突信息。如果`include_docs`非真则自动忽略。默认为false，只用于过滤查询。
* **descending** : Bool – 根据查询关键词进行逆序排序。默认为false，只用于过滤查询。
* **endkey** : String – 如果查到特定数据后，则停止继续查询。可选项。只用于过滤查询。
* **endkey_docid** : String – 如果查到特定文档编号后，则停止查询。可选项。只用于过滤查询。
* **include_docs** : Bool – 是否在查询中包括文档全文。默认为false。只用于过滤查询。
* **inclusive_end** : Bool – 说明查询终止字段是否包含在结果集内。默认为true。只用于过滤查询。
* **key** : String – 只返回与指定字段匹配的文档。可可选项。只用于过滤查询。
* **keys** : [String] – 只返回与指定数组内的字段匹配的文档。可选项。只用于过滤查询。
* **limit** : Int – 如果返回结果数量很大，则限制只返回指定数量的的结果。可选项。
* **skip** : Int – 返回结果前忽略若干条记录。默认为0。
* **stale** : CouchDBQueryStale – 允许从历史查询视图中读取记录，而不需要触发重建整个查询的操作。允许值为`ok`和`update_after`。可选项。默认为.ignore（忽略）。只适用于过滤查询。
* **startkey** : String – 返回自指定关键词匹配开始的记录。可选项，只适用于过滤查询。
* **startkey_docid** : String – 返回指定文档编号开始的记录。可选项，只是用于查询过滤。
* **update_seq** : Bool – 返回结果包括一个更新序列值 update_seq 。其取值反映出当前查询视图的序列号。默认为false，只适用于过滤查询。
* **selector** : [String:Any] – 用于控制筛选数据标准的JSON字典对象。更多内容参考选择器语法。只用于检索查询。
* **sort** : [String:Any] – 用于排序的字段名和值。可选项，只用于检索查询。
* **fields** : [String:Any] – 只有字典内的字段/取值才会作为结果返回。如果忽略的话，全部文档对象都会返回，而这个参数用于控制返回结果的内容形式。可选项，只用于检索查询。
* **use_index** : [String:Any] – 要求数据库按照特定索引进行查询。指定索引的形式可以是`<design\_document>` 或 `["<design\_document>", "<index\_name>"]`。可选项，只适用于检索查询。


## 错误响应

### enum CouchDBResponse

CouchDBResponse enum 枚举详细区别了执行CouchDB查询时可能出现的任何异常情况。

* **.undefined** : 没有定义
* **.ok** : 查询成功
* **.created** : 文档成功创建
* **.accepted** : 收到响应，但是预期的操作尚未完成。该情况常发生在后台操作，比如数据库压缩等等情况。
* **.notModified** : 额外的内容查询没有被修改。用于ETag标签系统，用于识别信息返回的版本。
* **.badRequest** : 错误的请求结构。这种错误通常意味着查询的URL地址、路径或者头数据有问题。如果内容的MD5编码校验后的结果不一致，则也会触发这个错误，用于提示数据传输有缺陷。
* **.unauthorized** : 未授权。如果没有足够的权限，或者权限不适用于某个资源，可能会引发该异常。
* **.forbidden** : 该请求内容或者操作被禁止。
* **.notFound** : 目标请求的内容无法找到。一般情况下如果还有更多提示信息，则会以一个JSON对象一同返回。该对象包括两个字段，即错误和原因，比如：`{"error":"not_found","reason":"no_db_file"}`
* **.resourceNotAllowed** : 请求使用了一个无效的类型。比如，预期资源要求使用POST，但是程序中却用了PUT；或者URL中包含了无效的字符串等，都会引发该异常。
* **.notAcceptable** : 服务器不支持这种类型的查询内容。
* **.conflict** : 查询导致了一个更新冲突。
* **.preconditionFailed** : 客户端查询的头数据与服务器容量不匹配。
* **.badContentType** : 内容的类型、请求的类型或提交的内容类型不受支持。
* **.requestedRangeNotSatisfiable** : 请求的头数据无法满足服务器约定的协议。
* **.expectationFailed** : 当发送文档块数据时，块数据传输失败。
* **.internalServerError** : 请求无效，或者因为有关的JSON格式不正确，或者请求的内容不正确。
