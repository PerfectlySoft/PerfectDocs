# PerfectHadoop: MapReduce Application 归并映射应用程序

该项目实现了Hadoop 归并映射应用程序 MapReduce Application Master REST API 的Swift类封装:

- `MapReduceApplication()`: 获取当前服务器上特定归并映射应用程序状态。

## 注意事项
⚠️本函数库为试验性模块，可能随时会有更新和调整。 ⚠️

## 连接到Hadoop映射归并应用程序服务器

为了能够连接到当前活动的Hadoop 映射归并应用程序，请首先用有效的 *应用程序代码* 初始化一个`MapReduceApplication()` 对象，并且需要包含必要参数：

``` swift
// 该连接可能只有基本操作权限
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com")
```
或者您还可以：

``` swift
// 如果有需要，增加用户名，用于执行更多有权限要求的操作
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com", user: "your user name")
```


### 用户身份认证
如果服务器需要使用 Kerberos 身份验证，请在代码中加入：

``` swift
// 将连接设置为KRB5认证模式
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com", user: "your user name", auth: .krb5)
```


### MapReduceApplication 映射归并应用程序对象属性
变量|类型|描述
----|---------|-----------
applicationId|String|应用程序编码。 *必选项* 
service|String|网站请求协议 - http 或 https
host|String|映射归并应用程序所在主机名称或者ip地址。
port|Int|映射归并应用程序所在主机使用的端口，默认为8088
auth| Authorization| 认证方法，.off（不认证）或者 .krb5（Kerberos V5版本验证方法）。默认是 .off，即不验证。
proxyUser|String|代理服务器用户名，如果适用的话
apibase|String|*只有* 当目标url应用程序界面接口api路径不是 `/ws/v1/mapreduce` 的时候，请填写该项目。
timeout|Int|超时（秒）。即访问服务器需要在该时间段内完成请求，否则网络会话过程自动断开并返回。如果设置为零则表示永远不超时（持续等待直到服务器返回）


## 查看基本信息

调用 `checkInfo()`方法可以获得当前映射归并主应用程序的基本运行情况，返回结果是一个 `MapReduceApplication.Info` 数据结构：

``` swift
guard let inf = try app.checkInfo() else {
	// 出错了
}
print(inf.startedOn)
print(inf.hadoopVersion)
print(inf.hadoopBuildVersion)
print(inf.hadoopVersionBuiltOn)
```

### MapReduceApplication.Info 数据结构

变量|类型|描述
----|---------|-----------
appId|Int|应用程序编码
startedOn|Int|应用程序启动时间（Unix纪元，毫秒）
name|string|应用程序名称
user|string|启动该程序的用户名
elapsedTime|long|自该应用程序启动后持续的时间（毫秒）

## MapReduce Job 映射归并作业

调用 `checkJobs()` 能够返回该应用程序的一个作业数组。
作业数组指的是目前该应用程序所执行的映射归并已完成的作业清单。返回的清单目前并未包含所有参数。最简单的形式就是直接调用 `checkJobs()` 而且不用包含任何参数，意味着无条件返回所有可以查询到的所有作业：


``` swift
let jobs = try app.checkJobs()
jobs.forEach { j in
  print(j.id)
  print(j.name)
  print(j.queue)
  print(j.state)
}
```

如果执行成功，`checkJobs()` 将返回一系列作业对象，如以下数据结构所述：

### 作业对象 Job 数据结构

变量|类型|描述
----|---------|-----------
id|String|作业编码
name|String|作业名称
queue|String|作业所在的队列名称
user|String|用户名
state|String|作业状态 - 有效值包括：NEW（新建）、INITED（初始化完成）、RUNNING（正在运行）、SUCCEEDED（执行成功）、FAILED（失败）、KILL_WAIT（关闭/终止等待）、KILLED（已经关闭/终止完成）、ERROR（出错）
startTime|Int|作业启动时间（Unix纪元，毫秒）
finishTime|Int|作业结束时间（Unix纪元，毫秒）
elapsedTime|Int|作业自启动以来的持续时间（毫秒）
mapsTotal|Int|总映射数
mapsCompleted|Int|已完成的映射总数
reducesTotal|Int|总归并数
reducesCompleted|Int|已完成的归并总数
diagnostics|String|诊断信息
uberized|Bool|当前作业是否是一个uber类型的作业——完全在应用程序主机上运行
mapsPending|Int|等待执行的映射数量
mapsRunning|Int|正在执行的映射数量
reducesPending|Int|等待执行的归并任务数量
reducesRunning|Int|正在执行的归并任务数量
failedReduceAttempts|Int|归并尝试失败的数量
killedReduceAttempts|Int|已经终止的归并尝试数量
successfulReduceAttempts|Int|成功完成的归并尝试数量
failedMapAttempts|Int|映射尝试失败的总数量
killedMapAttempts|Int|映射尝试被关闭/终止的总数量
successfulMapAttempts|Int|成功完成的映射尝试数量
acls|[ACL]|安全访问控制列表（ACL）集合

#### 访问控制列表对象 ACL 数据结构

变量|类型|描述
----|---------|-----------
name|String|安全访问列表名称
value|String|安全访问列表项的值


## 检查特定作业（Job）

可以通过作业代码（Job ID）检查特定作业信息：

``` swift
let job = try app.checkJob(jobId: "job_1484231633049_0005")
```

详见 [作业（Job）数据结构](# Job （作业）数据结构)。

## 作业尝试 JobAttempt

以下命令能够获得关于某个作业的尝试列表信息：

``` swift
guard let attempts = try app.checkJobAttempts(jobId: "job_1484231633049_0005") else {
	// 出错了
}
attempts.forEach { attempt in
	print(attempt.id)
	print(attempt.containerId)
	print(attempt.nodeHttpAddress)
	print(attempt.nodeId)
	print(attempt.startTime)
}
```

### JobAttempt 数据结构

变量|类型|描述
----|---------|-----------
id|String|作业尝试 (Job Attempt) 编码
nodeId|String|该尝试执行时所在的节点编号
nodeHttpAddress|String|该尝试执行时所在的节点地址
logsLink|String|作业尝试的日志超链接
containerId|String|该尝试执行时所在的容器编号
startTime|Long|该作业尝试启动的时间（Unix纪元，单位是毫秒）

## 作业指标统计（JobCounter）

使用作业指标函数库，您可以获得关于该作业的所有指标信息。


``` swift
guard let js = try app.checkJobCounters(jobId: "job_1484231633049_0005") else {
	// 出错了
}//end guard
js.counterGroup.forEach{ group in
  print(group.counterGroupName)
  group.counters.forEach { counter in
    print(counter.name)
    print(counter.mapCounterValue)
    print(counter.reduceCounterValue)
    print(counter.totalCounterValue)
  }//next counter
}//next group
```

### JobCounter Object

变量|类型|描述
----|---------|-----------
id|String|作业编码
counterGroup|[CounterGroup]|指标清单分组集合

#### CounterGroup 指标分组对象

变量|类型|描述
----|---------|-----------
counterGroupName|string|该组指标的名称
counter|[Counter]|指标对象构成的数组

#### Counter 指标对象

变量|类型|描述
----|---------|-----------
name|String|指标名称
reduceCounterValue|Int|归并任务计数
mapCounterValue|Int|映射任务计数
totalCounterValue|Int|所有任务总数统计

## 查看作业配置

函数 `checkJobConfig()` 可以用来检查作业配置情况：

``` swift
gurard let config = try app.checkJobConfig(jobId: "job_1484231633049_0005") else {
	/// 出错了
}
// print the configuration path
print(config.path)
// check properties of configuration file
for p in config.property {
  print(p.name)
  print(p.value)
  print(p.source)
}
```

### JobConfig 作业配置对象

作业配置对象包含了该作业的资源使用信息。

变量|类型|描述
----|---------|-----------
path|String|作业配置文件的路径
property|[Property]| 属性数组

#### Property 属性对象

变量|类型|描述
----|---------|-----------
name|String|配置项名称
value|String|配置项取值
source|String|配置对象所在路径。如果不止一个路径，则意味着占用资源的历史，清单末尾是最新的内容。

## 作业子任务（Tasks of A Job）

方法 `checkJobTasks()` 用于获取特定作业的子任务信息：

``` swift
// 获得特定作业的所有子任务信息
let tasks = try app.checkJobTasks(jobId: "job_1484231633049_0005")

// print properties of each task
for t in tasks {
  print(t.progress)
  print(t.elapsedTime)
  print(t.state)
  print(t.startTime)
  print(t.id)
  print(t.type)
  print(t.successfulAttempt)
  print(t.finishTime)
}//next t
```
### checkJobTasks() 函数参数

调用 `checkJobTasks()` 总是需要一个作业编码 jobId ，但是也可以指定第二个附加参数，即子任务类型用于进一步过滤查询结果：

- jobId: 作业编码
- taskType: 可选的子任务类型，即 `.MAP` 映射任务或者 `.REDUCE` 归并任务

### JobTask Object

函数`checkJobTasks()`用于返回一个作业子任务数组 `JobTask` 对象，包括下列属性：

变量|类型|描述
----|---------|-----------
id|string|作业子任务编码
state|String|作业子任务状态 - 有效值包括: NEW（新建）、SCHEDULED（已列入计划）、RUNNING（运行）、SUCCEEDED（成功）、FAILED（失败）、KILL_WAIT（等待终止/关闭）、KILLED（已经终止/关闭）
type|String|作业子任务类型 - MAP（映射）或者 REDUCE（归并）
successfulAttempt|String|最后一次成功尝试的作业子任务编码
progress|Double|作业子任务执行程度，百分比
startTime|Int|作业子任务启动时间（毫秒，Unix纪元）。如果还没有启动，则值为-1。
finishTime|Int|作业子任务结束时间（毫秒，Unix纪元）。
elapsedTime|Int|应用程序自启动至今所耗时间，单位是毫秒

## 查询特定的作业子任务 JobTask

如果已经获取了有效的 `jobId` 作业编码和 `jobTaskId` 作业子任务编码，那么可以使用函数 `checkJobTask()` 检查特定的作业子任务：

``` swift
guard let task = try app.checkJobTask(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
	// 出错了
}
print(task.progress)
```
该函数 `checkJobTask` 返回值为一个 [JobTask 作业子任务](### JobTask 作业子任务对象) 对象，如上所述。

## 作业子任务统计指标

方法 `checkJobTaskCounters()` 用于返回特定作业子任务的统计指标信息，如以下范例所示：

``` swift
guard let js = try app.checkJobTaskCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
	// 出错了
}
// 打印作业子任务指标编码
print(js.id)
// 检查每个统计指标编组
js.taskCounterGroup.forEach{ group in
  print(group.counterGroupName)
 // 打印每个分组中的详细指标信息
  group.counters.forEach { counter in
    print(counter.name)
    print(counter.value)
  }
}
```
### JobTaskCounter 作业子任务指标对象

变量|类型|描述
----|---------|-----------
id|String|作业编码
taskCounterGroup|[CounterGroup]|分组对象集合，详见 [CounterGroup 作业指标分组](#### CounterGroup 指标分组对象)

## Task Attempts 作业子任务尝试

函数 `checkJobTaskAttempts()` 用于检查特定作业子任务的尝试信息。

``` swift
let jobTaskAttempts = try app.checkJobTaskAttempts(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0")
for attempt in jobTaskAttempts {
  print(attempt.id)
  print(attempt.diagnostics)
  print(attempt.assignedContainerId)
  print(attempt.rack)
  print(attempt.state)
  print(attempt.progress)
}//next
```

一旦成功调用 `checkJobTaskAttempts()` 会返回一个 `TaskAttempt` 作业子任务尝试对象，如下所述：

### TaskAttempt 作业子任务尝试对象

作业子任务尝试对象 TaskAttempt 包括下列属性：

变量|类型|描述
----|---------|-----------
id|String|作业子任务编码
rack|String|所在机架信息
state|TaskAttempt.State|作业子任务尝试状态——有效值包括: NEW（新建）、UNASSIGNED（未指派）、 ASSIGNED（已指派）、RUNNING（运行）、COMMIT_PENDING（等待提交）、SUCCESS_CONTAINER_CLEANUP（成功并准备清理容器）、 SUCCEEDED（成功）、FAIL_CONTAINER_CLEANUP（清理容器失败）、FAIL_TASK_CLEANUP（作业子任务清理失败）、FAILED（失败）、KILL_CONTAINER_CLEANUP（终止清理容器）、KILL_TASK_CLEANUP、（终止作业子任务清理）、KILLED（已终止）
type|TaskType|作业子任务类型 - MAP 映射 或者 REDUCE 归并
assignedContainerId|String|该作业子任务尝试所在的容器编码
nodeHttpAddress|String|该作业子任务尝试坐在的http服务器节点地址
diagnostics|String|诊断信息
progress|Double|该作业子任务尝试的执行进度，百分比
startTime|Int|该作业子任务尝试启动时间（毫秒，Unix纪元）
finishTime|Int|该作业子任务尝试结束时间（毫秒，Unix纪元）
elapsedTime|Int|自该作业自任务尝试开始运行时所消耗时间（毫秒）


另外，对于 reduce （归并）作业子任务尝试来说，还可以包括下列属性：

变量|类型|描述
----|---------|-----------
shuffleFinishTime|Int|重新排序结束时间（毫秒，Unix纪元）
mergeFinishTime|Int|合并工作结束时间（毫秒，Unix纪元）
elapsedShuffleTime|Int|重新排序所耗时间（从归并子任务启动开始后到排序结束所用的时间）
elapsedMergeTime|Int|合并操作所用时间（从排序结束到合并完成总共的毫秒数）
elapsedReduceTime|Int|整个归并阶段所耗时间（从合并结束到整个归并子任务结束的总时间）


### 查询特定的 TaskAttempt 作业子任务尝试

如果作业子任务尝试编码有效，则可以执行`checkJobTaskAttempt()`查询该作业子任务的尝试信息，如下所述：

``` swift
guard let jobTaskAttempts = try app.checkJobTaskAttempt(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// 出错了
}//end guard
print(attempt.id)
print(attempt.diagnostics)
print(attempt.assignedContainerId)
print(attempt.rack)
print(attempt.state)
print(attempt.progress)
```
函数`checkJobTaskAttempt()` 调用后会返回一个“TaskAttempt 作业子任务尝试对象” 如上文所述。

## Attempt State Control 作业尝试状态控制

目前Hadoop 映射归并主程序具有一个试验性质的尝试状态控制。

### 检查尝试

如果需要查看尝试状态，请调用 `checkJobTaskAttemptState()` 方法：

``` swift
guard let state = try checkJobTaskAttemptState(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// 出错了
}//end guard
print(state)
```

### 终止尝试

调用 `killTaskAttempt()` 方法可以终止尝试：

``` swift
try killTaskAttempt(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0")
```

## 作业子任务尝试指标统计

方法 `checkJobTaskAttemptCounters()` 可以获得关于特定作业子任务尝试的详细指标统计，如下文所述：

``` swift
guard let counters = try app.checkJobTaskAttemptCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// 出错了
}
// 打印指标计数编码
print(counters.id)
// 遍历所有指标分组
for group in counters.taskAttemptCounterGroup {
  // 打印分组名称
  print(group.counterGroupName)
  // 打印该分组下的所有详细指标
  for counter in group.counters {
    print(counter.name)
    print(counter.value)
  }//next counter
}//next group
}

```
### JobTaskAttemptCounters 作业子任务尝试统计指标对象

变量|类型|描述
----|---------|-----------
id|String|作业编号
taskAttemptcounterGroup |[CounterGroup]|一个统计指标分组对象集合，详见[CounterGroup 统计指标分组](#### CounterGroup 指标分组对象)
