# Perfect-LDAP

该项目实现了一个对 OpenLDAP 的封装函数库，用于访问 OpenLDAP 和 Windows Active Directory 服务器。

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分。

请确保您已经安装并激活了最新版本的 Swift 4.0 tool chain 工具链。

*注意*: 由于LDAP在很多操作系统上都存在不同的服务器实现，因此本文中凡是标明了 (⚠️试验性质⚠️) 的方法，都以为着可能并不适用于某种场合。但是作为开源软件函数库，您可以随时根据需要修改源代码以达到目标要求。

## 快速上手

首先请在您的项目 Package.swift 文件中增加依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-LDAP.git", majorVersion: 3)
```

然后在源代码中导入 PerfectLDAP 函数库：

``` swift
import PerfectLDAP
```

## 连接到 LDAP 服务器

在连接到服务器时，您可以选择署名连接或者匿名连接。署名连接意味着您需要提供用户名密码之类的登陆信息，而匿名连接则不需要这些登录信息。完整的函数形式为： `LDAP(url:String, loginData: Login?, codePage: Iconv.CodePage)`. 其中 `codePage` 选项是为某些采用非UTF8编码方式的服务器而设置的。比如，如果使用`codePage: .GB2312`则可以连接到以简体中文为主要数据库编码的 LDAP 服务器。

### TLS 加密选项

PerfectLDAP 提供 TLS 加密选项，能够提高访问安全性。换句话说，连接的URL地址可以是 `ldap://` 或者 `ldaps://`，如下所示：

``` swift
// 连接到389端口，无编码加密
let ld = try LDAP(url: "ldap://perfect.com")
```
或者

``` swift
// 连接到636端口，证书加密
let ld = try LDAP(url: "ldaps://perfect.com")
```

### 连接超时

连接完成后，LDAP 对象可以设置超时属性，单位是秒：

``` swift
// 设置超时限制，以下例子将连接会话限制在10秒之内：
connection.timeout = 10
```

### 分阶段登录

许多服务器都要求执行操作前必须登录，但 PerfectLDAP 允许不同的登录方式，如下所示：

``` swift
// 以下代码展示了连接时同步登录。
// 注意当前线程会被锁住直到服务器返回或者超时。
// 首先准备登录信息：
let credential = LDAP.login( ... )
let connection = try LDAP(url: "ldaps://...", loginData: login)
```
除了以上同步连接暨登录方法之外，您还可以选择先连接后登录，实现多线程异步操作：

``` swift
// 首先创建连接
let connection = try LDAP(url: "ldaps:// ...")

// 准备登录信息
let credential = LDAP.login( ... )

// 采用异步方式登录
connection.login(info: credential) { err in
  // 注意正常情况下 err 是 nil ，如果 err 非空，则说明登录失败
}
```

## 登录方式选择

PerfectLDAP 使用一个`LDAP.Login`对象来实现不同的登录选项。各个不同的安全方式区别在于该对象的构造函数。

### 用户名密码登录

最简单的方法就是用用户名（DN）密码进行登录，调用构造函数 `LDAP.login(binddn: String, password: String)`即可，如下所示：

``` swift
let credential = LDAP.Login(binddn: "CN=judy,CN=Users,DC=perfect,DC=com", password: "0penLDAP")
```
### GSSAPI

如果需要使用GSSAPI进行身份验证，请调用`LDAP.login(user:String, mechanism: AuthType)`函数构造登录信息（前提是用户已经提前取得有效票据）:

``` swift
// 下列操作可以生成一个GSSAPI票据
let credential = LDAP.login(user: "judy", mechanism: .GSSAPI)
```

### GSS-SPNEGO 和 Digest-MD5 （⚠️试验性质⚠️)

对于其他 SASL 交互式登录机制，比如GSS-SPNEGO 和 Digest-MD5，请调用构造函数`LDAP.login(authname: String, user: String, password: String, realm: String, mechanism: AuthType)` 获取登录配置：

``` swift
// 将登录信息配置为DIGEST-MD5
let credential = LDAP.Login(authname: "judy", user: "DN:CN=judy,CN=Users,DC=perfect,DC=com", password: "0penLDAP", realm: "PERFECT.COM", mechanism: .DIGEST)
```
*⚠️注意⚠️* 参数 `authname` 等价于 `SASL_CB_AUTHNAME`，而 `user` 对应 `SASL_CB_USER`名称。如果您的程序不需要其中的某些参数，只要将该参数设置为空（“”）即可忽略。

## 检索

PerfectLDAP 提供了同步检索和异步检索两种方式，但函数参数都是一样的。

### 同步检索

同步检索将阻塞当前进程直到服务器返回或超时。完整的函数调用是`LDAP.search(base:String, filter:String, scope:Scope, attributes: [String], sortedBy: String) throws -> [String:[String:Any]]`，举例如下：

``` swift
// 执行同步检索并返回完整的属性集合，采用自然方式排序；返回结果为字典类型
let res = try connection.search(base: "CN=Users,DC=perfect,DC=com", filter:"(objectclass=*)")

print(res)
```

### 异步检索

异步检索将在单独的线程执行。线程结束时会调用回调函数，并将返回的查询结果传递给回调函数。完整的调用为 `LDAP.search(base:String, filter:String, scope:Scope, attributes: [String], sortedBy: String,  completion: @escaping ([String:[String:Any]])-> Void)`。和前面的例子等价的异步操作为：

``` swift
// 执行
// 执行异步检索。结束时回调并返回完整的属性集合，采用自然方式排序；返回结果为字典类型
connection.search(base: "CN=Users,DC=perfect,DC=com", filter:"(objectclass=*)") {
  res in
  print(res)
}
```

### 检索参数

- base: String, 检索时采用的基本DN，默认为空
- filter: String, 检索过滤条件，默认为 `"(objectclass=*)"`，即所有对象类
- scope: Scope, 检索范围，即 .BASE（基本） .SINGLE_LEVEL（单层） .SUBTREE（子树） 或 .CHILDREN（所有分支）
- sortedBy: String, 用于排序的字符串；可以用 `LDAP.sortingString()` 函数构造。
- completion: 回调函数，其返回参数为一个字典。如果字典为空则意味着查询失败。

#### 服务器端排序 （⚠️试验性质⚠️)
其中，`sortedBy` 参数是一个字符串，通知远程服务器按照该字符串的内容进行排序。但是您可以选择用下列方法构造该字符串，即用一个元组数组表示希望用一系列字段及其排序方法进行排序：

``` swift
// 每个元组包括两个部分：排序字段以及排序方法 - .ASC 升序或 .DSC 降序
let sort = LDAP.sortingString(sortedBy: [("description", .ASC)])
```

### 限制查询结果集

连接后，LDAP可以为查询结果设置限制选项`LDAP.limitation`。该整型变量用于说明每一个检索操作所能返回的最多记录数。

``` swift
// 设置检索返回结果集。本例中，每个检索只能返回前一千个记录
connection.limitation = 1000
```

## 属性操作

PerfectLDAP 为属性操作提供了增删改功能。每个操作都有同步和异步选项。

### 增加属性 （⚠️试验性质⚠️)

函数 `LDAP.add()` 可以针对一个特定 DN 增加属性，参数如下：

- distinguishedName: String, 目标 DN （唯一命名）
- attributes:[String:[String]], 以字典方式表达的属性集合。该字典中，每一个属性对应一个条目，每个条目之下都允许一个数组来保存多个字符串值。

无论同步add()还是异步add()都采用上面的参数，比如：

``` swift
// 同步增加属性
do {
  try connection.add(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["mail":["judy@perfect.com", "judy@perfect.org"]])
}catch (let err) {
    // 增加属性失败
}

// 异步增加属性
connection.add(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["mail":["judy@perfect.com", "judy@perfect.org"]]) { err in
  // 如果成功，err 应该是 nil
}
```

### 修改属性

函数 `LDAP.modify()` 可以针对一个特定 DN 修改属性，参数如下：

- distinguishedName: String, 目标 DN （唯一命名）
- attributes:[String:[String]], 以字典方式表达的属性集合。该字典中，每一个属性对应一个条目，每个条目之下都允许一个数组来保存多个字符串值。

无论同步修改还是异步修改都采用上面的参数，比如：

``` swift
// 同步修改属性
do {
  try connection.modify(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes: ["codePage":["437"]])
}catch (let err) {
    // 修改属性失败
}

// 异步修改属性
connection.modify(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com", attributes:["codePage":["437"]]) { err in
  // 如果成功，err 应该是 nil
}
```

### 删除属性（⚠️试验性质⚠️)

函数 `LDAP.delete()` 用于删除特定DN的所有属性，只有一个参数：

- distinguishedName: String, 目标 DN （唯一命名）

无论同步删除还是异步删除都采用上面的参数，比如：

``` swift
// 同步删除属性
do {
  try connection.delete(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com")
}catch (let err) {
    // 删除属性失败
}

// 异步删除属性
connection.delete(distinguishedName: "CN=judy,CN=User,DC=perfect,DC=com") { err in
  // 如果成功，err 应该是 nil
}
```

