# PerfectHadoop: MapReduce 历史服务器

本项目封装了MapReduce 历史服务器 REST API：

- `MapReduceHistory()`: 用于访问 MapReduce 历史服务器获取应用程序的 Swift 对象

## 连接到MapReduce历史服务器

如果希望连接到 Hadoop 的 Map Reduce 历史服务器，请初始化一个`MapReduceHistory()`对象，并确保参数准确无误：

``` swift
// 该连接只能保证基本的操作权限
let history = MapReduceHistory(host: "mapReduceHistory.somedomain.com", port: 19888)
```
或者连接到 Hadoop Map / Reduce 历史服务器时包括一个有效的用户名：

``` swift
// 如有需要，在此处增加用户名
let history = MapReduceHistory(host: "mapReduceHistory.somedomain.com", port: 19888, user: "your user name")
```

### 用户身份验证
如果您的服务器需要 Kerberos 验证，请增加身份验证选项：

``` swift
// 设置当前验证方式为 Kerberos
let history = MapReduceHistory(host: "mapReduceHistory.somedomain.com", port: 19888, user: "username", auth: .krb5)
```

### MapReduceHistory 对象参数详解
变量|类型|描述
---|---|-----------
service|String|Web 请求的类型 - http / https
host|String|目标Hadoop Map Reduce 历史服务器主机名或者IP地址
port|Int|默认的历史服务器端口，默认为 19888
auth|Authorization| 身份认证方式，.off 表示无要求（默认），.krb5 则表示采用Kerberos验证
proxyUser|String|代理服务器用户信息（可选项）
apibase|String|目标历史服务器的接口函数地址。⚠️*只有*⚠️在您的配置中其目标接口函数地址不是`/ws/v1/history`的时候才需要填写。
timeout|Int|连接超时限制，单位是秒。默认是零，也就是传输过程中始终保持等待。

## 获取服务器基本配置信息
调用 `checkInfo()` 能够获得历史服务器的基本信息，返回的数据将保存在一个 `MapReduceHistory.Info` 结构之内：

``` swift
guard let inf = try history.checkInfo() else {
	// 获取信息不成功
}
print(inf.startedOn)
print(inf.hadoopVersion)
print(inf.hadoopBuildVersion)
print(inf.hadoopVersionBuiltOn)
```

### `MapReduceHistory.Info` 数据结构

变量|类型|描述
----|---------|-----------
startedOn|Int|历史服务器启动时的时间戳（Unix纪元时间，单位是秒）
hadoopVersion|String| Hadoop 通用函数库的版本
hadoopBuildVersion|String|Hadoop 通用函数库编译信息，包括版本号，用户和校验码
hadoopVersionBuiltOn|String|Hadoop 通用函数库编译时间戳

## MapReduce 作业历史档案

调用 `checkJobs()` 可以返回一个以 `Job` 数据结构为元素的数组。
该数组列出了所有执行完毕的 MapReduce 作业清单；⚠️注意⚠️，受限于Hadoop的版本，目前暂时不能返回任务有关的所有参数。最简单的形式是调用 `checkJobs()`，不带任何参数即可。此时将返回所有变量:


``` swift
let jobs = try history.checkJobs()
// 如果不希望把有用的没用的都抓过来，可以通过设置下列参数筛选结果：
// let jobs = try his.checkJobs(state: .SUCCEEDED, queue: "default", limit: 10)
jobs.forEach { j in
  print(j.id) // 作业编号
  print(j.name) // 作业名称
  print(j.queue) // 作业队列
  print(j.state) // 作业状态
}
```

### checkJobs() 参数说明
变量|类型|描述
----|---------|-----------
**user** | String | 用户名
**state** | APP.FinalStatus | 作业状态 —— UNDEFINED（未定义）, SUCCEEDED（成功）, FAILED（失败） 和 KILLED（终止）
**queue** | String | 队列名称
**limit** | Int | 本次操作最多返回的任务数量
**startedTimeBegin** | Int | 从某个时刻之后开始的所有作业，采用Unix时间，单位是毫秒。
**startedTimeEnd** | Int | 从某个时间段内运行的所有作业（与startedTimeBegin配合），采用Unix时间，单位是毫秒。
**finishedTimeBegin** | Int | 所有在某个时间段内结束的作业，该参数为时间段的开始时间，采用Unix时间，单位是毫秒。
**finishedTimeEnd** | Int | 所有在某个时间段内结束的作业，该参数为时间段的结束时间，采用Unix时间，单位是毫秒。

### Job （作业）数据结构

变量|类型|描述
----|---------|-----------
id|String|作业编码
name|String|作业名称
queue|String|作业所提交到的目标队列
user|String|用户名
state|String|作业状态 - 有效值包括 NEW（新建）, INITED（完成初始化）, RUNNING（正在运行）, SUCCEEDED（执行成功）, FAILED（执行失败）, KILL_WAIT（正在结束处理）, KILLED（已被关闭）, ERROR（错误）
diagnostics|String|诊断信息
submitTime|Int|作业提交时刻（毫秒，Unix纪元）
startTime|Int|作业启动时刻（毫秒，Unix纪元）
finishTime|Int|作业结束时刻（毫秒，Unix纪元）
mapsTotal|Int|映射(map)总数
mapsCompleted|Int|已完成映射(map)数量
reducesTotal|Int|归并(reduce)总数
reducesCompleted|Int|已完成归并(reduce)数量
uberized|Boolean|该作业是否为uber类型——完全在Application Master应用程序主服务器上完成
avgMapTime|Int|每个分项映射作业的平均耗时（毫秒）
avgReduceTime|Int|每个分项归并作业的平均耗时（毫秒）
avgShuffleTime|Int|每个分项作业并行分派（shuffle）的平均耗时（毫秒）
avgMergeTime|Int|每个分享作业结果合并（merge）的平均耗时（毫秒）
failedReduceAttempts|Int|归并操作（reduce）失败次数统计
killedReduceAttempts|Int|归并操作（reduce）被终止的总数统计
successfulReduceAttempts|Int|归并操作（reduce）尝试成功的次数
failedMapAttempts|Int|失败的映射（map）尝试次数
killedMapAttempts|Int|被终止的映射（map）尝试次数
successfulMapAttempts|Int|成功的映射（map）尝试次数
acls|[ACL]|访问控制列表（ACL）数组

#### 访问控制列表对象 ACL 数据结构

变量|类型|描述
----|---------|-----------
name|String|安全访问列表名称
value|String|安全访问列表项的值

## 检查特定作业（Job）

可以通过作业代码（Job ID）检查历史服务器上的特定作业信息：

``` swift
let job = try history.checkJob(jobId: "job_1484231633049_0005")
```

详见 [作业（Job）数据结构](# Job （作业）数据结构)。

## 作业尝试 JobAttempt

以下命令能够获得关于某个作业的尝试列表信息：

``` swift
guard let attempts = try history.checkJobAttempts(jobId: "job_1484231633049_0005") else {
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
guard let js = try his.checkJobCounters(jobId: "job_1484231633049_0005") else {
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

### JobCounter 作业指标对象

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
guard let config = try his.checkJobConfig(jobId: "job_1484231633049_0005") else {
	/// 出错了
}
// 打印配置目录
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
let tasks = try his.checkJobTasks(jobId: "job_1484231633049_0005")

// 打印每个子任务的属性
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

### JobTask 作业子任务对象

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
guard let task = try his.checkJobTask(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
	// 出错了
}
print(task.progress)
```

该函数 `checkJobTask` 返回值为一个 [JobTask 作业子任务](### JobTask 作业子任务对象) 对象，如上所述。

## 作业子任务统计指标

方法 `checkJobTaskCounters()` 用于返回特定作业子任务的统计指标信息，如以下范例所示：

``` swift
guard let js = try his.checkJobTaskCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
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
let jobTaskAttempts = try his.checkJobTaskAttempts(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0")
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
guard let jobTaskAttempts = try his.checkJobTaskAttempt(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// 出错了
}//end guard
print(attempt.id)
print(attempt.diagnostics)
print(attempt.assignedContainerId)
print(attempt.rack)
print(attempt.state)
print(attempt.progress)
```

函数`checkJobTaskAttempt()` 调用后会返回一个 [TaskAttempt 作业子任务尝试对象](### TaskAttempt 作业子任务尝试对象) 如上文所述。

## 作业子任务尝试指标统计

方法 `checkJobTaskAttemptCounters()` 可以获得关于特定作业子任务尝试的详细指标统计，如下文所述：

``` swift
guard let counters = try his.checkJobTaskAttemptCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
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

```


