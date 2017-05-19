# PerfectHadoop: YARN 节点管理器

本项目提供一个YARN节点管理器 REST API的Swift类封装：

- `YARNNodeManager()`: 用于访问在一个YARN节点管理器上所有应用程序和容器信息的类实例

 
## 连接到YARN节点管理器

采用Perfect连接到Hadoop YARN 节点管理器最简单的方法，就是初始化一个`YARNNodeManager()`对象并提供足够参数：

``` swift
// 以下连接可能只提供基本操作权限
let nm = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042)
```
或者包含一个有效的用户名连接到Hadoop YARN 节点管理器：

``` swift
// 如有必要请自行修改用户名
let nm = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042, user: "your user name")
```

### 用户身份认证

如果您的服务器正在使用Kerberos，请参考以下程序代码：

``` swift
// 将认证模式设置为krb5
let yarn = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042, user: "username", auth: .krb5)
```

### YARNNodeManager 节点管理器对象初始化参数

变量|类型|描述
----|---------|-----------
service|String|服务器支持的协议 - http / https
host|String|目标YARN节点管理器的主机名/域名或者ip地址
port|Int|目标主机端口，默认为 8042
auth|Authorization| 用户身份验证机制，.off表示不验证（默认），.krb5为Kerberos V5 验证方法
proxyUser|String|代理服务器用户，如果有的话
apibase|String|*只有*在您的服务器编程接口API不是 `/ws/v1/node` 的情况下才需要设置
timeout|Int|连接超时设置，单位是秒。零表示传输永远不超时，保持等待直到服务器返回结果

## 获取服务器基本信息

调用 `checkOverall()` 方法可以获得服务器基本信息，返回结果为一个 `YARNNodeManager.Info` 数据结构

``` swift
guard let inf = try yarn.checkOverall() else {
	// 出错了
}
print(inf.id)
print(inf.nodeHostName)
print(inf.healthReport)
```

### YARNNodeManager.NodeInfo 数据结构

变量|类型|描述
----|---------|-----------
id|String|节点管理器编号
nodeHostName|String|节点管理器主机名称
totalPmemAllocatedContainersMB|Int|为容器分配的实际物理内存总量，兆字节
totalVmemAllocatedContainersMB|Int|为容器分配的虚拟内存总量，兆字节
totalVCoresAllocatedContainers|Int|为容器分配的虚拟内核数量
lastNodeUpdateTime|Int|最后一次收到健康报告的时间戳（Unix纪元，毫秒）
healthReport|String|节点健康报告详细诊断书
nodeHealthy|Bool|节点是否健康，真为健康，假为不健康
nodeManagerVersion|String|节点管理器版本号
nodeManagerBuildVersion|String|节点管理器编译信息，包括编译版本、用户和校验和nodeManagerVersionBuiltOn|String|节点管理器编译时间戳（Unix纪元，毫秒）
hadoopVersion|String|当前Hadoop Common（通用库）的版本
hadoopBuildVersion|String|当前Hadoop Common（通用库）的版本，包括编译子版本、用户和校验和
hadoopVersionBuiltOn|String|当前Hadoop Common（通用库）编译时的时间戳（Unix纪元，毫秒）

## 应用程序

调用 `checkApps()` 方法可返回一个 `APP` 应用程序对象组成的数组。每个应用程序对象都是代表这个应用程序的一组资源集合。

``` swift
let apps = try yarn.checkApps()
apps.forEach { a in
  print(a.id)
  print(a.containerids)
  print(a.state)
}//next
```

### 应用程序数据结构

变量|类型|描述
----|---------|-----------
id|String|应用程序编号
user|String|启动该应用程序的用户名
state|APP.State|应用程序状态 - 有效值包括：NEW（新建）、INITING（正在初始化）、RUNNING（正在运行）、FINISHING_CONTAINERS_WAIT（运行结束，容器正在等待结果）、 APPLICATION_RESOURCES_CLEANINGUP（应用程序资源正在清理）、FINISHED（运行结束）
containerids|[String]]|当前在该节点上应用程序所使用的容器编号列表。如果为空则表示当前在该节点上没有针对该应用程序的容器

## 查看特定应用程序

如果具有有效的应用程序代号，您可以调用`checkApp()`来查询节点信息：

``` swift
guard let app = try yarn.checkApp("application_1326121700862_0005") else {
	// 出错了
}
// print out all container ids of the application
print(app.containerids)
```
`checkApp()` 会返回一个 `APP` 对象，如前文所述

## 检查容器

调用 `checkContainers()` 可以查看当前节点上的所有容器：

``` swift
// 获取所有容器
let containers = try ynode.checkContainers()

// 查看容器数量
print(containers.count)

containers.forEach { container in
	print(container.id)
	print(container.state)
	print(container.nodeId)
	print(container.diagnostics)
	/// ...
}

```

`checkContainers()` 返回值时一个容器数组 `Container`如下所述：

### 容器对象

变量|类型|描述
----|---------|-----------
id|String|容器编号
state|String|容器状态 - 有效状态包括： NEW（新建）、LOCALIZING（正在本地化）、 LOCALIZATION_FAILED（本地化失败）、LOCALIZED（本地化完成）、RUNNING（正在运行）、 EXITED_WITH_SUCCESS（退出，结果成功）、EXITED_WITH_FAILURE（退出，结果失败）、KILLING（正在关闭／终止）、CONTAINER_CLEANEDUP_AFTER_KILL（容器在终止/关闭后进行清理）、CONTAINER_RESOURCES_CLEANINGUP（容器在正常结束后进行资源清理）、DONE（完成）
nodeId|String|容器所在节点编号
containerLogsLink|String|容器日志的http链接
user|String|启动该容器的用户名
exitCode|Int|容器退出的返回值
diagnostics|String|出错容器的诊断信息
totalMemoryNeededMB|Int|容器所需的内存总量，兆字节
totalVCoresNeeded|Int|容器所需的虚拟内核数量

## 查看特定容器

如果具有有效的容器编号，可以使用 `checkContainer()` 检查特定容器：

``` swift
guard let container = try ynode.checkContainer(id: container.id) else {
	// 出错了
}
print(container.id)
print(container.state)
print(container.nodeId)
print(container.diagnostics)
```

该方法会返回一个`Container`对象，如前文所述。
