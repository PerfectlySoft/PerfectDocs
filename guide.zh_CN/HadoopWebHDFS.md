# PerfectHadoop: WebHDFS

该项目实现了一个对 WebHDFS 网络接口的封装。

## 快速上手

### 连接到 Hadoop 集群

如果希望通过 webhdfs 连接到 HDFS 集群，请初始化一个 WebHDFS 类对象并包含必要参数：

``` swift
// 以下连接方式只能提供基本操作功能，比如浏览
let hdfs = WebHDFS(host: "hdfs.somedomain.com", port: 9870)
```
或者增加有效的用户名：

``` swift
// add user name to do more operations such as modifiction of file or directory
let hdfs = WebHDFS(host: "hdfs.somedomain.com", port: 9870, user: "username")
```

#### 身份认证
如果目标 Hadoop 集群采用 Kerberos 认证机制，请增加以下参数：

``` swift
// set auth to kerberos
let hdfs = WebHDFS(host: "hdfs.somedomain.com", port: 9870, user: "username", auth: .krb5)
```

#### WebHDFS 对象参数说明
- `service`: 协议字符串，比如 - http / https / webhdfs / hdfs
- `host`: 主机名或 IP 地址
- `port`: webhdfs 主机端口，默认为 9870
- `auth`: 认证模式，默认是 `.off`
- `proxyUser`: 代理用户，如果需要的话
- `apibase`: 如果目标服务器用的不是标准 /webhdfs/v1/ 路径协议，则在此设置。
- `timeout`: 超时设置。默认情况下所有WebHDFS操作都会阻塞当前进程，因此设置该参数允许秒级等待，一旦超时则解除阻塞立即返回。

### 获取根目录
调用函数 `getHomeDirectory()` 以返回用户根目录：

``` swift
let home = try hdfs.getHomeDirectory()
print("用户根目录是： \(home)")
```

### 获取文件状态
调用`getFileStatus()`方法可以返回一个`FileStatus`结构，该结构包含以下属性：

#### `FileStatus` 数据结构
- `accessTime`: Int, 最近访问时间（unix时间戳)
- `pathSuffix`: String, 文件名的后缀（类型）
- `replication`: Int, 冗余存储节点数量
- `type`: String, 节点类型：目录还是文件
- `blockSize`: Int, 存储块尺寸，标准为 128MB，最小为 1MB
- `owner`: String, 节点宿主用户名
- `modificationTime`: Int, 最近修改时间（unix时间戳）
- `group`: String, 节点用户群组信息
- `permission`: Int, 节点权限，标准的unix权限格式： (u)rwx (g)rwx (o)rwx
- `length`:Int, 文件长度

以下代码演示了如何获取文件状态`getFileStatus()`：

``` swift
let fs = try hdfs.getFileStatus(path: "/")
if fs.length > 0 {
	...
}
```

### 文件列表
函数 `listStatus()` 会返回一个`[FileStatus]`数组，用于列出目录下所有文件及其属性：

``` Swift
let list = try hdfs.listStatus(path: "/")
for file in list {
	// 打印每个文件的宿主用户
	print(file.owner)
}
```
每个文件的状态结构与 `getFileStatus()` 返回的结构相同。


### 创建目录
基本 HDFS 目录操作包括`mkdir()` 和 `delete()`。为了创建一个名为"/demo"的目录，并设置其初始权限为 754（ rwxr-xr-- ，也就是用户可以读写执行、组用户可读可执行、其他用户只读），请参考以下代码:

``` swift
try hdfs.mkdir(path: "/demo", permission: 754)
```

### 目录统计
WebHDFS 提供目录统计功能 `getDirectoryContentSummary()`，展开后的统计项明细如下：

#### `ContentSummary` 结构属性
- `directoryCount`: Int, 子目录数量
- `fileCount`: Int, 文件数量
- `length`: Int, 节点长度
- `quota`: Int, 节点限额
- `spaceConsumed`: Int, 节点当前占用的空间
- `spaceQuota`: Int, 节点空间限额
- `typeQuota`: 三个`Quota`（用量与限额）结构，每个结构都包含两个整型字段： `consumed`（已用） and `quota`（限额）:
  - `ARCHIVE`: Quota, 归档文件用量与限额信息
  - `DISK`: Quota, 存储在磁盘上的用量和限额信息
  - `SSD`: Quota, 存储在固态硬盘上的用量和限额信息

请调用 `getDirectoryContentSummary()` 方法获取有关文件（目录）的上述详细信息，参考以下代码：

``` swift
let sum = try hdfs.getDirectoryContentSummary(path: "/")
print(sum.length)
print(sum.spaceConsumed)
print(sum.typeQuota.SSD.consumed)
print(sum.typeQuota.SSD.quota)
print(sum.typeQuota.DISK.consumed)
print(sum.typeQuota.DISK.quota)
print(sum.typeQuota.ARCHIVE.consumed)
print(sum.typeQuota.ARCHIVE.quota)
...
```
### 文件校验

文件校验`getFileCheckSum()`方法提供目标文件的校验信息`FileCheckSum`结构，该结构包含以下三个属性：

#### `FileChecksum` 结构属性
- `algorithm`: String, 当前校验的累加和算法
- `bytes`: String, 累加和字符串
- `length`: Int, 累加和长度

参考下面的程序检查文件的完整性：

``` swift
let checksum = try hdfs.getFileCheckSum(path: "/book/chickenrun.txt")
// 文件累加和算法
print(checksum.algorithm)
// 累加和字符串
print(checksum.bytes)
// 累加和长度
print(checksum.length)
```

### 删除文件或目录
下面的程序展示了如何使用`delete()`函数删除目录或文件。如果要删除目录，还可以设置其中的`recursive`属性用于递归删除（即删除所有子目录）

``` swift
// 删除文件
try hdfs.delete(path "/demo/boo.txt")

// 删除目录及所有子目录
try hdfs.delete(path:"/demo", recursive: true)
```

### 上传文件
上传文件请调用 `create()` 方法，需要至少包括本地文件名和远程的目标目录名称，比如：

``` swift
try hdfs.create(path: "/目标目录", localFile: "/tmp/本地文件.txt")
```
考虑到上传文件是一个耗时的操作，请自行结合超时设置和线程进行上传操作。

#### 参数
方法`create()`的参数包括：

- `path`:String, 远程文件的完整路径
- `localFile`:String, 本地文件的完整路径
- `overwrite`:Bool, 如果远程文件已存在，是否覆盖
- `permission`: unix风格文件属性 (u)rwx (g)rwx (o)rwx。默认为755 —— rwxr-xr-x
- `blocksize`:Int, 每个文件块的尺寸。缺省为 128M，最小值为 1M
- `replication`:Int, 冗余备份的节点数量。忽略该参数系统将按照 hdfs 默认值进行设置
- `buffersize`: 传输文件时的缓冲区大小。忽略该参数系统将按照 hdfs 默认值进行设置

### 符号链接
同Unix系统一样，HDFS 提供一个方法叫做`createSymLink`，用于为目录或者文件创建一个符号链接：

``` swift
try hdfs.createSymLink(path: "/book/真文件.txt", destination:"/我的/最近的项目/链接.lnk", createParent: true)
```
请⚠️注意⚠️其中有一个参数叫做`createParent`，意思是在创建过程中如果没有响应的路径，则系统会自动把对应路径一并创建。

### 文件下载
请使用 `openFile()` 方法下载文件内容

``` swift
let (text, bytes) = try hdfs.openFile(path: "/书刊/睡前故事.txt")
print(bytes.count)
```
上述例子中，文件"睡前故事.txt"会被同时保存到二进制数组 `bytes`中。

因为下载文件通常是一个耗时的操作，请考虑采用多线程异步的方式进行调用。在处理大文件下载时，您还可以针对同一个文件多次调用 `openFile()`方法，实现大文件分段下载。这种方式可以保证在某个文件片段出错后可以进行二次下载，确保文件整体下载内容无误。

#### Parameters
- `path`:String, 远程文件完整路径
- `offset`:Int, 读取文件的起始位置偏移量
- `length`:Int, 希望读取的字节数
- `buffersize`:Int, 传输用的缓冲区大小

### 文件追加
文件追加的操作与创建差不多，但不是覆盖，而是将本地文件上传后追加到原文件结尾：

``` swift
try hdfs.append(path: "/远程文件", localFile: "/tmp/本地文件.txt")
```
#### Parameters
- - `path`:String, 远程文件完整路径
- `localFile`:String, 本地文件：待上传的本地文件
- `buffersize`:Int, 文件传输用缓冲区大小
### 文件合并
HDFS 允许用户将多个文件合并到一起：

``` swift
try hdfs.concat(path:"/tmp/1.txt", sources:["/tmp/2.txt", "/tmp/3.txt"])
```

上面的例子里，文件 2.txt 和 3.txt 会按顺序追加到 1.txt 之后。

### 截取文件
HDFS 文件可以通过下列函数进行截短：

``` swift
try hdfs.truncate(path: "/书刊/西游记.txt", newlength: 1024)
```
如果操作成功，则文件的新长度只剩下 1K。

### 设置权限

HDFS 文件权限的设置方法是`setPermission()`. 以下例子将目录“/demo”的权限设置为了 754， 也就是754（ rwxr-xr-- ，也就是用户可以读写执行、组用户可读可执行、其他用户只读）：

``` swift
try hdfs.setPermission(path: "/demo", permission: 754)
```

### 设置宿主用户
`setOwner()` 用于将文件宿主设置为其他用户：

``` swift
try hdfs.setOwner(path: "/书刊/小鸡快跑.html", name:"新读者", group: "新群组")
```

### 设置冗余备份数量
HDFS 文件系统允许为每个文件设置一个以上的冗余备份。请使用函数`setReplication()`实现该工作：

``` swift
try hdfs.setReplication(path: "/书刊/侠客行.txt", factor: 2)
```

### 设置访问时间和修改时间
HDFS 允许改变文件的最后访问时间或最近修改时间，格式为unix时间戳。参考以下例子实现类似 unix 的 touch 命令操作：

``` swift
// 取得当前 unix 时间戳
let now = time(nil)
try hdfs.setTime(path: "/tmp/某个文件.txt", modification: now, access: now)
```

### 访问控制列表
HDFS 文件系统访问控制列表可以通过以下函数进行修改：

- `getACL`: 获取访问控制列表
- `setACL`: 设置访问控制列表
- `modifyACL`: 修改访问控制列表
- `removeACL`: 删除一个或多个访问列表

`getACL()` 方法会返回一个 `AclStatus`结构，内容如下：

- `entries`: [String], ACL 清单
- `owner`: String, 宿主用户信息
- `group`: String, 所在用户群信息
- `permission`: Int, unix 样式权限
- `stickyBit`: Bool, 标记位。如果sticky位设置，则为真

以下程序展示了如何进行访问控制列表操作：

``` swift
let hdfs = WebHDFS(auth:.byUser(name: defaultUserName))
let remoteFile = "/acl.txt"
do {
	// 获取访问控制表
	var acl = try hdfs.getACL(path: remoteFile)
	print("所在组信息：\(acl.group)")
	print("宿主用户信息：\(acl.owner)")
	print("控制清单：\(acl.entries)")
	print("访问权限：\(acl.permission)")
	print("标记：\(acl.stickyBit)")

	try hdfs.setACL(path: remoteFile, specification: "user::rw-,user:hadoop:rw-,group::r--,other::r--")

	try hdfs.modifyACL(path: remoteFile, entries: "user::rwx,user:hadoop:rwx,group::rwx,other::---")

   try hdfs.removeACL(path: remoteFile, defaultACL: false)
	try hdfs.removeACL(path: remoteFile)
	try hdfs.removeACL(path: remoteFile, entries: "", defaultACL: false)
```

### 检查命令权限
函数`checkAccess()`用于检验当前用户是否具备某个命令的执行权限：

``` swift
let b = try hdfs.checkAccess(path: "/", fsaction: "mkdir")
// true value means user can perform mkdir() on the root folder
if b {
	print("mkdir: 授权操作")
} else {
	print("mkdir: 未授权操作")
}
```

### 扩展属性
除了传统的文件属性之外，HDFS 还能够提供可自行定制的更多文件属性，称为扩展属性XAttr，操作包括：

- `setXAttr`: 设置扩展属性
- `getXAttr`: 获得一个或多个扩展属性
- `listXAttr`: 列出所有属性
- `removeXAttr`: 删除一个或多个属性

此外，XAttr 扩展属性还包括两个标志：`CREATE`（创建属性） and `REPLACE`（覆盖数值），在设置扩展属性时默认标志为 CREATE，详见如下定义：

``` swift
public enum XAttrFlag:String {
	case CREATE = "CREATE"
	case REPLACE = "REPLACE"
}
```

请参考以下代码

``` swift
let remoteFile = "/book/a.txt"

try hdfs.setXAttr(path: remoteFile, name: "user.color", value: "red")
// 如果成功，则文件将增加新属性'user.color'，值为红色

try hdfs.setXAttr(path: remoteFile, name: "user.size", value: "small")
// 如果成功，则文件将增加新属性'user.size'，值为小

try hdfs.setXAttr(path: remoteFile, name: "user.build", value: "2016")
// 如果成功，则文件将增加新属性'user.build'，值为2016

try hdfs.setXAttr(path: remoteFile, name: "user.build", value: "2017", flag:.REPLACE)
// 注意参数标识flag，这里用了REPLACE，意味着扩展属性user.build的值将被替换为2017

// 打印所有的扩展属性
let list = try hdfs.listXAttr(path: remoteFile)
list.forEach {
	item in
	print(item)
}//next

// 获取特定的扩展属性    
var a = try hdfs.getXAttr(path: remoteFile, name: ["user.color", "user.size", "user.build"])
// 打印获取到的属性
a.forEach{
	x in
	print("\(x.name) => \(x.value)")
}//next

try hdfs.removeXAttr(path: remoteFile, name: "user.size")
// 如果成功，则属性中的user.size 将被删除

```

### 快照
HDFS 提供目录快照功能，如`createSnapshot()`、`renameSnapshot()` 和 `deleteSnapshot`。

- `CreateSnapshot()`

如果成功，函数`createSnapshot()` 将返回一个组合结果`(长名称, 短名称)`。长名称就是路径全称，短名称为快照自己的名字，参考如下命令：

```swift
let (fullpath, shortname) = try hdfs.createSnapshot(path: "/mydata")
// 路径全称
print(fullpath)
// 快照名称
print(shortname)
```

- `renameSnapshot()`
快照重命名：

``` swift
try hdfs.renameSnapshot(path: "/mydata", from: shortname, to: "snapshotNewName")
// 如果成功则快照名称被改变为snapshotNewName
```

- `deleteSnapshot()`

用 `deleteSnapshot()` 删除快照：

``` swift
try hdfs.deleteSnapshot(path: dir, name: shortname)
// 如果成功，则表示快照被删除
```
