# PerfectHadoop: YARN 资源管理器

本项目提供了YARN资源管理器的Swift类封装：

- `YARNResourceManager()`: 用于访问YARN的集群信息，包括簇及其性能参数、调度器和程序提交情况。


## 连接到 YARN 资源管理器

只需要初始化`YARNResourceManager()`对象并提供必要参数即可连接到YARN资源管理器：

``` swift
// 以下连接仅支持基本操作
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088)
```
比如提供一个有效的用户名：

``` swift
// 增加用户名
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088, user: "your user name")
```

### 用户身份验证

如果您的服务器在使用 Kerberos 验证机制，请增加参数如下：

``` swift
// 将认证机制设置为KRB5：
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088, user: "username", auth: .krb5)
```

### YARNResourceManager 资源管理器对象数据结构

变量|类型|描述
----|---------|-----------
service|String|服务器支持的协议 - http / https
host|String|目标YARN资源管理器的主机名/域名或者ip地址
port|Int|目标主机端口，默认为 8088
auth|Authorization| 用户身份验证机制，.off表示不验证（默认），.krb5为Kerberos V5 验证方法
proxyUser|String|代理服务器用户，如果有的话
apibase|String|*只有*在您的服务器编程接口API不是 `/ws/v1/cluster` 的情况下才需要设置。
timeout|Int|连接超时设置，单位是秒。零表示传输永远不超时，保持等待直到服务器返回结果

## 获取服务器基本信息

调用 `checkClusterInfo()` 方法可以获得服务器基本信息，返回结果为一个 `ClusterInfo` 数据结构：

``` swift
guard let i = try yarn.checkClusterInfo() else {
	// 出错了
}//end guard
print(i.startedOn)
print(i.state)
print(i.hadoopVersion)
print(i.resourceManagerVersion)

```
调用成功后`checkClusterInfo()` 会返回 `ClusterInfo` 集群对象信息，如下所述：

### ClusterInfo 集群信息对象

变量|类型|描述
----|---------|-----------
id|Int|集群代号
startedOn|Int|集群启动的时间（Unix纪元，毫秒）
state|String|资源管理器状态 —— 有效值包括：NOTINITED（尚未初始化）、INITED（已经初始化）、STARTED（已经启动）、STOPPED（已经停止）
haState|String|资源管理器 HA （高可靠性状态） - 有效值包括： INITIALIZING（正在初始化）、ACTIVE（活动）、STANDBY（等待）、STOPPED（已经停止）
resourceManagerVersion|String|资源管理器版本号
resourceManagerBuildVersion|String|资源管理器编译信息，包括编译版本、用户和校验和
resourceManagerVersionBuiltOn|String|资源管理器编译时的时间戳（Unix纪元，毫秒）
hadoopVersion|String|当前Hadoop Common（通用库）的版本
hadoopBuildVersion|String|当前Hadoop Common（通用库）的版本，包括编译子版本、用户和校验和
hadoopVersionBuiltOn|String|当前Hadoop Common（通用库）编译时的时间戳（Unix纪元，毫秒）

## 集群性能计量

调用方法 `checkClusterMetrics()` 可以返回集群详细性能指标信息：

``` swift
guard let m = try yarn.checkClusterMetrics() else {
	// something wrong
}
print(m.availableMB)
print(m.availableVirtualCores)
print(m.allocatedVirtualCores)
print(m.totalMB)
```


一旦调用成功， `checkClusterMetrics()` 会返回如下的集群性能指标对象：

### ClusterMetrics 集群性能指标对象

变量|类型|描述
----|---------|-----------
appsSubmitted|Int|已提交的应用程序数量
appsCompleted|Int|已完成的应用程序数量
appsPending|Int|等待执行的应用程序数量
appsRunning|Int|正在执行的应用程序数量
appsFailed|Int|执行失败的应用程序数量
appsKilled|Int|被关闭/终止的应用程序数量
reservedMB|Int|内存保有量（预留但未使用），单位是兆字节
availableMB|Int|内存可用量，单位是兆字节
allocatedMB|Int|已分配内存总量，单位是兆字节
totalMB|Int|内存总量，单位是兆字节
reservedVirtualCores|Int|虚拟内核保有数量（预留但未使用）
availableVirtualCores|Int|虚拟内核可用数量
allocatedVirtualCores|Int|已分配的虚拟内核数量
totalVirtualCores|Int|虚拟内核总数
containersAllocated|Int|已分配的容器数量
containersReserved|Int|容器保有量（预留但未使用）
containersPending|Int|等待分配的容器数量
totalNodes|Int|节点总数量
activeNodes|Int|活动节点总数
lostNodes|Int|掉线节点数
unhealthyNodes|Int|不健康节点数
decommissionedNodes|Int|已取消任务的节点数
rebootedNodes|Int|已经重启的节点数

## 集群调度信息

函数 `checkSchedulerInfo()` 用于返回集群调度详细信息

``` swift
guard let sch = try yarn.checkSchedulerInfo() else {
	// 出错了
}
print(sch.capacity)
print(sch.maxCapacity)
print(sch.queueName)
print(sch.queues.count)
```

如果调用成功， `checkSchedulerInfo()` 会返回一个 `SchedulerInfo` 数据结构，如下所示：

### SchedulerInfo 调度信息对象

变量|类型|描述
----|---------|-----------
availNodeCapacity|Int|可用节点容量
capacity|Double|与其父队列相比，已配置的队列容量百分比。
maxCapacity|Double|队列最大容量
maxQueueMemoryCapacity|Int|与其父队列相比，已配置的队列最大容量百分比。
minQueueMemoryCapacity|Int|队列内存最低容量
numContainers|Int|容器数量
numNodes|Int|节点总数量
qstate|QState|队列状态 - 有效值包括：STOPPED（停止）、RUNNING（正在运行）
queueName|String|队列名称
queues| [Queue]|队列集合
rootQueue| FairQueue|根队列集合
totalNodeCapacity|Int|节点总容量
type|String|调度器类型 - capacityScheduler 容量调度器
usedCapacity|Double|已使用的队列容量，用百分比表示
usedNodeCapacity|Int|已使用的节点数量，用百分比表示

#### FairQueue 根队列对象

变量|类型|描述
----|---------|-----------
maxApps|Int|当前队列容许的应用程序最大数量
minResources|ResourcesUsed|已配置的、承诺提供给该队列的最低资源总量
maxResources|ResourcesUsed|已配置的、允许提供给该队列的最高资源总量
usedResources|ResourcesUsed|在队列内已分配给容器的资源总和
fairResources|ResourcesUsed|根队列内共享的资源数
clusterResources|ResourcesUsed|集群容量
queueName|String|队列名称
schedulingPolicy|String|当前队列使用的调度策略
childQueues|FairQueue|子队列集合信息。如果队列没有子队列则忽略
type|String|队列类型 - fairSchedulerLeafQueueInfo
numActiveApps|Int|当前队列内活动的应用程序数量
numPendingApps|Int|队列内等待执行的应用程序数量

#### 队列对象

变量|类型|描述
----|---------|-----------
absoluteCapacity|Double|队列可以使用的资源数量与整个集群资源总量的百分比
absoluteMaxCapacity|Double|队列可以使用的最大资源数量与整个集群资源总量的百分比
absoluteUsedCapacity|Double|队列当前正在使用的最大资源数量与整个集群资源总量的百分比
capacity|Double|相对于父队列来说，已配置的资源百分比。
maxActiveApplications|Int|当前队列内允许的活动应用程序最大数量
maxActiveApplicationsPerUser|Int|当前队列内每个用户允许的活动应用程序最大数量
maxApplications|Int|当前队列允许装载的应用程序最大数量
maxApplicationsPerUser|Int|当前队列内每个用户允许装载的应用程序最大数量
maxCapacity|Double|相对于父队列来说当前队列允许配置的最大容量百分比
numActiveApplications|Int|当前队列中活动的应用程序数量
numApplications|Int|当前队列中的应用程序数量
numContainers|Int|正在使用的容器数量
numPendingApplications|Int|当前队列中等待执行的应用程序数量
queueName|String|队列名称
queues|[Queue]|子队列集合信息。如果队列没有子队列则忽略
resourcesUsed|ResourcesUsed|当前队列使用的资源数量总和
state|String|队列状态
type|String|队列类型 - capacitySchedulerLeafQueueInfo
usedCapacity|Double|队列已经使用的容量百分比
usedResources|String|当前队列正在使用的资源，字符串
userLimit|Int|在配置中用户用量下限百分比
userLimitFactor|Double|在配置中的用户限额因子（放大系数）
users|[User]|用户对象数组，每个用户对象都包含了当前资源的用量，如下所示：

#### 用户对象

变量|类型|描述
----|---------|-----------
username|String|占用当前资源的用户名
resourcesUsed|ResourcesUsed |当前用户所占资源用量，如下定义
numActiveApplications|Int|当前队列中该用户旗下活动的应用程序数量
numPendingApplications|Int|当前队列中该用户旗下等待调度的应用程序数量

#### ResourcesUsed 已占用资源

变量|类型|描述
----|---------|-----------
memory|int|每个容器所需要的内存总量
vCores|int|每个容器所需要的虚拟内核数量

## 集群节点

### 查看所有节点

调用方法 `checkClusterNodes()` 能够以数组形式返回一个集群中的所有节点信息。

``` swift
let nodes = try yarn.checkClusterNodes()
nodes.forEach { node in
	print(node.rack)
	print(node.availableVirtualCores)
	print(node.availMemoryMB)
	print(node.healthReport)
	print(node.healthStatus)
	print(node.id)
	print(node.lastHealthUpdate)
	print(node.nodeHostName)
	print(node.nodeHTTPAddress)
```

调用完成后，`checkClusterNodes()` 返回的节点对象数据结构如以下章节所述：

#### 节点对象

变量|类型|描述
----|---------|-----------
rack|String|该节点所在机架位置信息
state|String|节点状态 - 有效值为：NEW（新建）、RUNNING（运行中）、UNHEALTHY（不健康）、DECOMMISSIONED（任务取消）、LOST（丢失）、REBOOTED（已经重启）
id|String|节点编号
nodeHostName|String|节点主机名称
nodeHTTPAddress|String|节点的HTTP地址
healthStatus|String|节点健康状态 - 健康或者不健康
healthReport|String|详细的健康报告
lastHealthUpdate|Int|最后一次节点报告为健康的时间点（Unix纪元，毫秒）
usedMemoryMB|Int|当前节点使用内存总量（兆字节）
availMemoryMB|Int|当前节点可用内存总量（兆字节）
usedVirtualCores|Int|当前节点在用虚拟内核数量
availableVirtualCores|Int|当前节点可用虚拟内核总数
numContainers|int|当前节点容器数量

### Check A Node

调用方法 `checkClusterNode()`可获得具体节点的详细数据结构。

``` swift
guard let n = try yarn.checkClusterNode(id: "host.domain.com:8041") else {
	// 出错了
}
```

## 集群应用程序

### 查看所有应用程序

调用方法 `checkApps()` 可以获得集群上所有应用程序：

``` swift
let apps = try yarn.checkApps()
// 或者可以增加下列查询参数以缩小查询范围：
/// let apps = try yarn.checkApps(states: [APP.State.FINISHED, APP.State.RUNNING], finalStatus: APP.FinalStatus.SUCCEEDED)

apps.forEach{ a in
	print(a.allocatedMB)
	print(a.allocatedVCores)
	print(a.amContainerLogs)
	print(a.amHostHttpAddress)
	print(a.amNodeLabelExpression)
	print(a.amRPCAddress)
	print(a.applicationPriority)
	print(a.applicationTags)
}
```

一旦成功调用， `checkApps()` 方法会返回一个应用程序对象组成的数组，每个对象的数据结构说明如下：

#### 应用程序对象

变量|类型|描述
----|---------|-----------
id|String|应用程序编号
user|String|启动该程序的用户名
name|String|应用程序名称
applicationType|String|应用程序类型
queue|String|提交应用程序的队列
state|String|应用程序的状态（相对于资源管理器而言）- 有效值包括（自YarnApplicationStatus枚举）：NEW（新建）、NEW_SAVING（正在保存新建应用）、SUBMITTED（已经提交）、ACCEPTED（已经接受）、RUNNING（正在运行）、FINISHED（运行结束）、FAILED（运行失败）、KILLED（应用程序已经被终止/关闭）
finalStatus|String|如果应用程序已经结束，则该属性为应用程序的最终状态 - 由应用程序自身汇报而来 - 有效值包括：UNDEFINED（未定义）、SUCCEEDED（成功）、FAILED（失败）、KILLED（被关闭／终止）
progress|Double|应用程序执行完成度百分比
trackingUI|String|跟踪程序当前指向的URL链接类型 - History（历史服务器）或者是 ApplicationMaster（应用程序主服务器）
trackingUrl|String|跟踪程序当前指向的URL链接
diagnostics|String|详细诊断信息
clusterId|Int|集群编号
startedTime|Int|应用程序启动时间（Unix纪元，毫秒）
finishedTime|Int|应用程序结束时间（Unix纪元，毫秒）
elapsedTime|Int|自应用程序启动开始已经经历的毫秒数
amContainerLogs|String|应用程序主容器日志的URL链接
amHostHttpAddress|String|应用程序主服务器的节点http地址
amRPCAddress|String|应用程序主服务器的 RPC 远程过程调用地址
allocatedMB|Int|当前应用程序所在运行容器的已分配内存总数
allocatedVCores|Int|当前应用程序所在运行容器的已分配虚拟内核总数
runningContainers|Int|当前运行该应用程序的容器数量
memorySeconds|Int|应用程序所分配到的内存总数（兆字节／秒）
vcoreSeconds|Int|该应用程序已分配的CPU资源数量（虚拟内核／秒）
unmanagedApplication|Bool|当前应用程序是否在管控之下
applicationPriority|Int|应用程序提交优先级
appNodeLabelExpression|String|节点标签表达式，用于在节点上区别哪一个容器会成为运行该程序的默认容器
amNodeLabelExpression|String|节点标签表达式，用于在节点上区别哪一个容器会成为运行该程序期望运行的容器

### 查看特定应用程序

调用方法 `checkApp()` 可以用于查看特定应用程序：

``` swift
let a = try yarn.checkApp(id: "application_1484231633049_0025")
print(a.allocatedMB)
print(a.allocatedVCores)
print(a.amContainerLogs)
print(a.amHostHttpAddress)
print(a.amNodeLabelExpression)
print(a.amRPCAddress)
print(a.applicationPriority)
print(a.applicationTags)

```

返回值为应用程序结构，如上文所述。

### 申请一个新应用程序

调用方法 `newApplication()` 可以返回一个新建的应用程序句柄。

``` swift
guard let a = try yarn.newApplication() else {
	// 新建应用程序失败
}
```

一旦调用成功，该方法 `newApplication()` 会返回一个 NewApplication 新程序对象，如下所述：

#### NewApplication 新程序对象

变量|类型|描述
----|---------|-----------
id|String|新建应用程序编号
maximumResourceCapability|ResourcesUsed|该集群上可以使用的最大资源数量

#### ResourcesUsed 资源用量数据结构

新建的 `NewApplication` 对象会包含一个名为 `ResourceUsed` 的特殊对象，如下所述：

变量|类型|描述
----|---------|-----------
memory|Int|容器内容许的最大内存数量开销
vCores|Int|容器内容许的最大虚拟内核数量开销


### 修改新建程序模板并正式提交为应用程序

在新建应用程序基础上，修改该程序的具体内容并调用 `submit()` 方法能够将修改结果提交为一个可以真正运行的应用程序。

⚠️注意⚠️ Yarn映射归并应用程序提交的详细方法已经超过了本文的范围，详细请参考[下一代 Hadoop 映射归并程序——自定义YARN应用 https://wiki.apache.org/hadoop/WritingYarnApps](https://wiki.apache.org/hadoop/WritingYarnApps)

``` swift

// 首先创建一个空应用程序模板
let sum = SubmitApplication()

// *必须* 设置应用程序编号
sum.id =  "application_1484231633049_0025"
// 设置应用程序名称
sum.name = "test"

// 分配空白的本地资源
let local = LocalResource(resource: "hdfs://localhost:9000/user/rockywei/DistributedShell/demo-app/AppMaster.jar", type: .FILE, visibility: .APPLICATION, size: 43004, timestamp: 1405452071209)

// 将资源信息指定到一个数组
let localResources = Entries([Entry(key:"AppMaster.jar", value: local)])

// *必须* 填写应用程序的映射命令
let commands = Commands("/hdp/bin/hadoop jar /hdp/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha1.jar grep input output 'dfs[a-z.]+'")

// 设置环境变量
let environments = Entries([Entry(key:"DISTRIBUTEDSHELLSCRIPTTIMESTAMP", value: "1405459400754"), Entry(key:"CLASSPATH", value:"{{CLASSPATH}}<CPS>./*<CPS>{{HADOOP_CONF_DIR}}<CPS>{{HADOOP_COMMON_HOME}}/share/hadoop/common/*<CPS>{{HADOOP_COMMON_HOME}}/share/hadoop/common/lib/*<CPS>{{HADOOP_HDFS_HOME}}/share/hadoop/hdfs/*<CPS>{{HADOOP_HDFS_HOME}}/share/hadoop/hdfs/lib/*<CPS>{{HADOOP_YARN_HOME}}/share/hadoop/yarn/*<CPS>{{HADOOP_YARN_HOME}}/share/hadoop/yarn/lib/*<CPS>./log4j.properties"), Entry(key:"DISTRIBUTEDSHELLSCRIPTLEN", value:6), Entry(key:"DISTRIBUTEDSHELLSCRIPTLOCATION", value: "hdfs://localhost:9000/user/rockywei/demo-app/shellCommands")])

// 设置容器信息
sum.amContainerSpec = AmContainerSpec(localResources: localResources, environment: environments, commands: commands)

// 设置其他信息
sum.unmanagedAM = false
sum.maxAppAttempts = 2
sum.resource = ResourceRequest(memory: 1024, vCores: 1)
sum.type = "MapReduce"
sum.keepContainersAcrossApplicationAttempts = false

// 设置日志环境
sum.logAggregationContext = LogAggregationContext(logIncludePattern: "file1", logExcludePattern: "file2", rolledLogIncludePattern: "file3", rolledLogExcludePattern: "file4", logAggregationPolicyClassName: "org.apache.hadoop.yarn.server.nodemanager.containermanager.logaggregation.AllContainerLogAggregationPolicy", logAggregationPolicyParameters: "")

// 设置验证失败间隔（1小时）
sum.attemptFailuresValidityInterval = 3600000

// 如果可能的话，设置资源预留代码
sum.reservationId = "reservation_1454114874_1"
sum.amBlackListingRequests = AmBlackListingRequests(amBlackListingEnabled: true, disableFailureThreshold: 0.01)

try yarn.submit(application: sum)
```

### SubmitApplication 待提交应用程序对象

`SubmitApplication` 类包括了一系列子类以完成应用程序的内容组成，详见下文：

变量|类型|描述
----|---------|-----------
id|String|应用程序编号
name|String|应用程序名称
queue|String|该应用程序准备提交的目的队列名称
priority|Int|该应用程序优先级
amContainerSpec|AmContainerSpec|应用程序目标主容器环境，如下文所述
unmanagedAM|Bool|当前应用程序是否属于一个未管制的应用程序主服务器
maxAppAttempts|int|该应用程序最大重试次数
resource|ResourceRequest|应用程所申请的资源数量，如下文所述
type|String|应用程序类型(MapReduce、Pig、Hive等等)
keepContainersAcrossApplicationAttempts|Bool|是否需要YARN资源管理器保留该应用程序的所有容器（在运行结束后）或者删除这些容器
tags|[String]|应用程序的标签列表。关于如何使用标签，请参考本文后续示范代码
logAggregationContext|LogAggregationContext|所有节点管理器（NodeManager）所需要的所有用于处理该应用程序信息的日志集合
attemptFailuresValidityInterval|Int|最多失败次数。一旦超过该数值，则后续失败次数将不计入失败总数
reservationId|string|代表调度器内相应预留资源的唯一名称编号
amBlackListingRequests| AmBlackListingRequests |请求黑名单，比如 “允许/禁用 应用程序主服务器黑名单管理” 和 “禁止失败限制”


#### AmContainerSpec 应用程序容器指标对象

变量|类型|描述
----|---------|-----------
localResources|Entries|描述需要进行本地化的资源清单，如下说明
environment|Entries|容器的环境变量列表，“环境项目变量名“与”值”成对出现
commands|Commands|启动容器的命令行，按照输入命令的先后顺序执行
serviceData|Entries|应用程序所需要的附加服务数据；项目名称为所期望的附加服务名称，值是base64编码的、用于传递给附加服务程序的自定义数据
credentials|Credentials|应用程序运行所需密码，如下文所述
applicationAcls| Entries |应用程序所需安全控制列表；项目名称可以是 “VIEW_APP”（查看应用程序）或者 “MODIFY_APP” （修改应用程序），对应的值是用户名单及配套权限

#### LocalResource 本地资源对象

变量|类型|描述
----|---------|-----------
resource|String|待本地化资源的具体位置
type|ResourceType|资源类型，可选项为“ARCHIVE”（存档）、“FILE”（文件）和 “PATTERN”（符合某种模式的资源）
visibility|Visibility|本地化资源的可见性；可选项为“PUBLIC”（公开资源）、“PRIVATE”（私有资源）以及“APPLICATION”（应用程序独占资源）
size|Int|待本地化的资源大小
timestamp|Int|待本地化资源的时间戳

#### Commands 命令对象

目前 Commands 命令对象只有一个名为 `command` 的字段。
⚠️**NOTE**⚠️ 根据 Hadoop 3.0 alpha 参考手册，commands 应该为一组命令行组成的字符串数组，但是该参考手册中给出的范例却只有一个有效的字符串作为命令行，因此该功能为试验性，有可能根据情况随时升级。

#### Credentials 密码对象

变量|类型|描述
----|---------|-----------
tokens|[String:String]|希望传递给应用程序的令牌，以字典形式声明。每个条目是令牌的项目名称，而值是令牌数据，用于使用对应的Web服务时获取
secrets|[String:String]|应用程序中会使用到的密码，同样是字典形式声明；条目是变量名称，值是base64编码字符串用于密码加密

#### ResourceRequest 资源请求对象

变量|类型|描述
----|---------|-----------
memory|int|每个容器所需要的内存总量
vCores|int|每个容器所需要的虚拟内核数量

#### Entries 字典对象

`Entries` 对象只有一个元素`entry`，可以是一个`Entry`普通字典或者`EncryptedEntry`加密字典。 每个 `Entry` 有一对表达式组成： `key`是条目名称，`value`是对应的值，类型是`Any`任意类型。

而两种不同字典 `Entry` 普通字典和 `EncryptedEntry` 加密字典的区别是`EncryptedEntry` 会在提交到应用程序前进行base64加密，只要值的类型是 `String` 字符串或者`[UInt8]`二进制数组。

#### AmBlackListingRequests 应用程序主服务器黑名单请求

变量|类型|描述
----|---------|-----------
amBlackListingEnabled|Bool|在应用程序主服务器上启动或者禁用请求黑名单
disableFailureThreshold|Float|应用程序主服务器黑名单条件限额（即如果某个应用程序反复失败次数达到一定程度时，应被列为请求黑名单，之后任何针对该应用的重新提交操作都会被忽略）

#### LogAggregationContext 日志集合环境对象

变量|类型|描述
----|---------|-----------
logIncludePattern|String|当应用程序结束时，如果其日志内容符合本变量定义的某种模式，则会被上传到日志服务器
logExcludePattern|String|当应用程序结束时，如果其日志内容不符合本变量定义的某种模式，则会被上传到日志服务器
rolledLogIncludePattern|String|当应用程序结束时，如果其日志内容符合本变量定义的某种模式，则会被上传到日志服务器，并采用滚动的方式进行保存和覆盖
rolledLogExcludePattern|String|当应用程序结束时，如果其日志内容不符合本变量定义的某种模式，则不会并采用滚动的方式进行保存和覆盖
logAggregationPolicyClassName|String|说明NodeManager节点管理器在处理日志时所应用的策略（Policy）名称
logAggregationPolicyParameters|String|需要传递给策略对象（Policy）的参数

## 应用程序状态控制

调用方法 `getApplicationStatus()` 会返回当前应用程序的状态。

``` swift
guard let state = try yarn.getApplicationStatus(id:  "application_1484231633049_0025") else {
	// 出错了
}
print(state)
```

方法 `setApplicationStatus()` 用于将应用程序的状态调整为预期设置。

``` swift
try yarn.setApplicationStatus(id:  "application_1484231633049_0025", state: .KILLED)
```

有效的状态包括：NEW（新建）、NEW_SAVING（正在保存新建应用）、SUBMITTED（已经提交）、ACCEPTED（已经接受）、RUNNING（正在运行）、FINISHED（运行结束）、FAILED（运行失败）、KILLED（应用程序已经被终止/关闭）

## 应用程序队列控制

调用方法 `getApplicationQueue()` 可返回当前应用程序队列名称：

``` swift
guard let queue = try yarn.getApplicationQueue(id:  "application_1484231633049_0025") else {
	// 出错了
}
print(queue)
```

方法 `setApplicationQueue()` 则可用于设置当前队列的名称。

``` swift
try yarn.setApplicationQueue(id:  "application_1484231633049_0025", queue:"a1a")
```

## 应用程序优先级控制

调用方法 `getApplicationPriority()` 可以返回当前应用程序的优先级

``` swift
guard let priority = try yarn.getApplicationPriority(id:  "application_1484231633049_0025") else {
	// 出错了
}
print(priority)
```

当前优先级返回结果是一个整数，比如零。

调用方法 `setApplicationPriority()`能用于设置当前应用程序的优先级：

``` swift
try yarn.setApplicationPriority(id:  "application_1484231633049_0025", priority: 1)
```

## 应用程序尝试

调用方法 `checkAppAttempts()` 可以返回一组应用程序的尝试信息：

``` swift
guard let attempts = try yarn.checkAppAttempts(id:  "application_1484231633049_0025") else {
	// 出错了
}
attempts.forEach { attempt in
	print(attempt.containerId)
	print(attempt.id)
	print(attempt.nodeHttpAddress)
	print(attempt.nodeId)
	print(attempt.startTime)
}//next
```

一旦成功调用 `checkAppAttempts()` 会返回 AppAttempt 应用程序尝试对象数组，其数据结构如下所示：

### AppAttempt 应用程序尝试对象

变量|类型|描述
----|---------|-----------
id|String|该尝试的编码
nodeId|String|该尝试所在的节点编号
nodeHttpAddress|String|该尝试所在节点的http地址
logsLink|String|该尝试的日志链接
containerId|String|该尝试所在的容器编号
startTime|Int|该尝试的启动时间（Unix纪元，毫秒）

## 应用程序统计数据

调用方法 `checkAppStatistics()` 可以返回一组应用程序的统计指标变量。

``` swift
let sta = try yarn.checkAppStatistics(states: [APP.State.FINISHED, APP.State.RUNNING])
sta.forEach{ s in
	print(s.count)
	print(s.state)
	print(s.type)
}//next s
```

`checkAppStatistics()` 函数允许两个额外参数：

变量|类型|描述
---------|---------|-----------
states|[APP.State]|需要进入统计的应用程序状态。如果忽略本参数，则该方法将返回所有应用程序无论状态如何。
applicationTypes|[String]|需要进入统计的应用程序类型。如果忽略本参数，则该方法会返回所有应用程序无论类型如何。在这种情况下，返回结果会在应用程序类型上显示通配符`*`。注意目前仅支持唯一的应用程序类型就是映射归并，比如 `applicationTypes: ["MapReduce"]`。
