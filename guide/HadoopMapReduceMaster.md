# PerfectHadoop: MapReduce Application

This project provides a Swift wrapper of MapReduce Application Master REST API:

- `MapReduceApplication()`: access a specific map / reduce application running on current server.

## RELEASE NOTE
⚠️This is an experimental module and is subject to change in future. ⚠️

## Connect to Hadoop Map Reduce Application

To connect to a current active Hadoop Map / Reduce Application by Perfect, initialize a `MapReduceApplication()` object with an *APPLICATION ID*, and other essential parameters:

``` swift
// this connection could possibly do some basic operations
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com")
```
you can also 

``` swift
// add user name if need
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com", user: "your user name")
```


### Authentication
If using Kerberos to authenticate, please try codes below:

``` swift
// set auth to kerberos
let app = MapReduceApplication(applicationId: "application_12345678_xxxxxx", host: "mapReducer.somedomain.com", user: "your user name", auth: .krb5)
```


### Parameters of MapReduceApplication Object

Item|Data Type|Description
----|---------|-----------
applicationId|String|the id string of the application to control. *Required* 
service|String|the service protocol of web request - http / https
host|String|the hostname or ip address of the Hadoop Map Reduce Application Master host
port|Int|the port of map reduce application master host, default is 8088
auth| Authorization| .off or .krb5. Default value is .off
proxyUser|String|proxy user, if applicable
apibase|String|use this parameter *ONLY* the target server has a different api routine other than `/ws/v1/mapreduce`
timeout|Int|timeout in seconds, zero means never timeout during transfer


## Get General Information
Call `checkInfo()` to get the general information of a Hadoop MapReduce Application Master in form of a `MapReduceApplication.Info` structure:

``` swift
guard let inf = try app.checkInfo() else {
	// something goes wrong here
}
print(inf.startedOn)
print(inf.hadoopVersion)
print(inf.hadoopBuildVersion)
print(inf.hadoopVersionBuiltOn)
```

### Members of `MapReduceApplication.Info `

Item|Data Type|Description
----|---------|-----------
appId|Int|The application id
startedOn|Int|The time the application started (in ms since epoch)
name|string|The name of the application
user|string|The user name of the user who started the application
elapsedTime|long|The time since the application was started (in ms)

## MapReduce Jobs

Call `checkJobs()` to return an array of `Job` structure.
The jobs resource provides a list of the MapReduce jobs that have finished. It does not currently return a full list of parameters. The simplest form is `checkJobs()`, which will return all jobs available:


``` swift
let jobs = try app.checkJobs()
jobs.forEach { j in
  print(j.id)
  print(j.name)
  print(j.queue)
  print(j.state)
}
```
If succeeded, `checkJobs()` would probably return a job object as described below:

### Data Structure of Job

Item|Data Type|Description
----|---------|-----------
id|String|The job id
name|String|The job name
queue|String|The queue the job was submitted to
user|String|The user name
state|String|the job state - valid values are: NEW, INITED, RUNNING, SUCCEEDED, FAILED, KILL_WAIT, KILLED, ERROR
startTime|Int|The time the job started (in ms since epoch)
finishTime|Int|The time the job finished (in ms since epoch)
elapsedTime|Int|The elapsed time since job started (in ms)
mapsTotal|Int|The total number of maps
mapsCompleted|Int|The number of completed maps
reducesTotal|Int|The total number of reduces
reducesCompleted|Int|The number of completed reduces
diagnostics|String|A diagnostic message
uberized|Bool|Indicates if the job was an uber job - ran completely in the application master
mapsPending|Int|The number of maps still to be run
mapsRunning|Int|The number of running maps
reducesPending|Int|The number of reduces still to be run
reducesRunning|Int|The number of running reduces
failedReduceAttempts|Int|The number of failed reduce attempts
killedReduceAttempts|Int|The number of killed reduce attempts
successfulReduceAttempts|Int|The number of successful reduce attempts
failedMapAttempts|Int|The number of failed map attempts
killedMapAttempts|Int|The number of killed map attempts
successfulMapAttempts|Int|The number of successful map attempts
acls|[ACL]|A collection of acls objects

#### Elements of the acls object
Item|Data Type|Description
----|---------|-----------
value|String|The acl value
name|String|The acl name

## Check a Specific Job 
It is also possible to check a specific job by a job id:

``` swift
let job = try app.checkJob(jobId: "job_1484231633049_0005")
```

See [Data Structure of Job](# Data Structure of Job) as above.

## JobAttempt of Job
When you make a request for the list of job attempts, the information will be returned as an array of job attempt objects.

``` swift
guard let attempts = try app.checkJobAttempts(jobId: "job_1484231633049_0005") else {
	// something goes wrong
}
attempts.forEach { attempt in
	print(attempt.id)
	print(attempt.containerId)
	print(attempt.nodeHttpAddress)
	print(attempt.nodeId)
	print(attempt.startTime)
}
```

### JobAttempt Structure:
Item|Data Type|Description
----|---------|-----------
id|String|The job attempt id
nodeId|String|The node id of the node the attempt ran on
nodeHttpAddress|String|The node http address of the node the attempt ran on
logsLink|String|The http link to the job attempt logs
containerId|String|The id of the container for the job attempt
startTime|Long|The start time of the attempt (in ms since epoch)

## Counters of Job
With the job counters API, you can object a collection of resources that represent al the counters for that job.

``` swift
guard let js = try app.checkJobCounters(jobId: "job_1484231633049_0005") else {
	// something wrong
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

Item|Data Type|Description
----|---------|-----------
id|String|The job id
counterGroup|[CounterGroup]|An array of counter group objects

#### CounterGroup Object

Item|Data Type|Description
----|---------|-----------
counterGroupName|string|The name of the counter group
counter|[Counter]|An array of counter objects

#### Counter Object
Item|Data Type|Description
----|---------|-----------
name|String|The name of the counter
reduceCounterValue|Int|The counter value of reduce tasks
mapCounterValue|Int|The counter value of map tasks
totalCounterValue|Int|The counter value of all tasks

## Check Configuration of Job
Use `checkJobConfig()` to check configuration of a specific job:

``` swift
gurard let config = try app.checkJobConfig(jobId: "job_1484231633049_0005") else {
	/// something wrong
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

### JobConfig Object
A job configuration resource contains information about the job configuration for this job.

Item|Data Type|Description
----|---------|-----------
path|String|The path to the job configuration file
property|[Property]| an array of property object

#### Property Object
Item|Data Type|Description
----|---------|-----------
name|String|The name of the configuration property
value|String|The value of the configuration property
source|String|The location this configuration object came from. If there is more then one of these it shows the history with the latest source at the end of the list.

## Tasks of a Job

Use `checkJobTasks()` to collect task information from a specific job:

``` swift
// get all tasks information from a specific job
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
### Parameters of checkJobTasks()

Calling `checkJobTasks()` requires a jobId all the time, however, you can also specify an extra parameter to indicate what type of tasks shall be listed in the query results:

- jobId: a string represents the id of the job
- taskType: optional, could be `.MAP` or `.REDUCE`

### JobTask Object

The `checkJobTasks()` will return an array of `JobTask` data structure, as described below:

Item|Data Type|Description
----|---------|-----------
id|string|The task id
state|String|The state of the task - valid values are: NEW, SCHEDULED, RUNNING, SUCCEEDED, FAILED, KILL_WAIT, KILLED
type|String|The task type - MAP or REDUCE
successfulAttempt|String|The id of the last successful attempt
progress|Double|The progress of the task as a percent
startTime|Int|The time in which the task started (in ms since epoch) or -1 if it was never started
finishTime|Int|The time in which the task finished (in ms since epoch)
elapsedTime|Int|The elapsed time since the application started (in ms)

## Query a Specific JobTask

If both `jobId` and `jobTaskId` are available, you can use `checkJobTask()` to perform a query on a specific job task:

``` swift
guard let task = try app.checkJobTask(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
	// something wrong
}
print(task.progress)
```
The return value of `checkJobTask` will be a [JobTask](# JobTask Object) object, as described above.

## Task Counters of Job

Method `checkJobTaskCounters()` can return counters of a specific task, as the example below:

``` swift
guard let js = try app.checkJobTaskCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0") else {
	// something wrong
}
// print the id of the job task counters
print(js.id)
// check out each counter group
js.taskCounterGroup.forEach{ group in
  print(group.counterGroupName)
  // print counters in each group.
  group.counters.forEach { counter in
    print(counter.name)
    print(counter.value)
  }
}
```
### JobTaskCounter Object

Item|Data Type|Description
----|---------|-----------
id|String|The job id
taskCounterGroup|[CounterGroup]|An array of counter group objects, See [CounterGroup](# CounterGroup Object)

## Task Attempts

Function `checkJobTaskAttempts()` can return attempts from a specific job task.

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

Once done, the `checkJobTaskAttempts` will return an array of `TaskAttempt` object, as described below:

### TaskAttempt Object

Elements of the TaskAttempt object:

Item|Data Type|Description
----|---------|-----------
id|String|The task id
rack|String|The rack
state|TaskAttempt.State|The state of the task attempt - valid values are: NEW, UNASSIGNED, ASSIGNED, RUNNING, COMMIT_PENDING, SUCCESS_CONTAINER_CLEANUP, SUCCEEDED, FAIL_CONTAINER_CLEANUP, FAIL_TASK_CLEANUP, FAILED, KILL_CONTAINER_CLEANUP, KILL_TASK_CLEANUP, KILLED
type|TaskType|The type of task - MAP or REDUCE
assignedContainerId|String|The container id this attempt is assigned to
nodeHttpAddress|String|The http address of the node this task attempt ran on
diagnostics|String|A diagnostics message
progress|Double|The progress of the task attempt as a percent
startTime|Int|The time in which the task attempt started (in ms since epoch)
finishTime|Int|The time in which the task attempt finished (in ms since epoch)
elapsedTime|Int|The elapsed time since the task attempt started (in ms)


Besides, For reduce task attempts you also have the following fields:

Item|Data Type|Description
----|---------|-----------
shuffleFinishTime|Int|The time at which shuffle finished (in ms since epoch)
mergeFinishTime|Int|The time at which merge finished (in ms since epoch)
elapsedShuffleTime|Int|The time it took for the shuffle phase to complete (time in ms between reduce task start and shuffle finish)
elapsedMergeTime|Int|The time it took for the merge phase to complete (time in ms between the shuffle finish and merge finish)
elapsedReduceTime|Int|The time it took for the reduce phase to complete (time in ms between merge finish to end of reduce task)


### Query a Specific TaskAttempt
If a taskAttempt id is available, you can also check the specific one by `checkJobTaskAttempt()`, as demo below:

``` swift
guard let jobTaskAttempts = try app.checkJobTaskAttempt(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// something wrong
	print("there is no such a task attempt")
	return
}//end guard
print(attempt.id)
print(attempt.diagnostics)
print(attempt.assignedContainerId)
print(attempt.rack)
print(attempt.state)
print(attempt.progress)
```
The `checkJobTaskAttempt()` will return such an [TaskAttempt object](# TaskAttempt Object) as described above.

## Attempt State Control

Currently Hadoop MapReduce Application Master has an experimental attempt state control.

### Check Attempt
To check a running attempt state, use `checkJobTaskAttemptState()`:
``` swift
guard let state = try checkJobTaskAttemptState(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// something wrong
	print("there is no such a task attempt")
	return
}//end guard
print(state)
```
### Kill Attempt
To terminate an attempt, call `killTaskAttempt()`:

``` swift
try killTaskAttempt(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0")
```

## Attempt Counters of Task

Method `checkJobTaskAttemptCounters()` can return counters of a specific task attempt, as the example below:

``` swift
guard let counters = try app.checkJobTaskAttemptCounters(jobId: "job_1484231633049_0005", taskId: "task_1326381300833_2_2_m_0", "attempt_1326381300833_2_2_m_0_0") else {
	// something wrong
	print("Task Attempt Counters Acquiring failed")
	return 
}
// print the counters' id
print(counters.id)
// enumerate all groups
for group in counters.taskAttemptCounterGroup {
  // print group name
  print(group.counterGroupName)
  // enumerate all counters in this group
  for counter in group.counters {
    print(counter.name)
    print(counter.value)
  }//next counter
}//next group
}

```
### JobTaskAttemptCounters Object

Item|Data Type|Description
----|---------|-----------
id|String|The job id
taskAttemptcounterGroup |[CounterGroup]|An array of counter group objects, See [CounterGroup](# CounterGroup Object)
