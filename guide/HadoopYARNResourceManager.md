# PerfectHadoop: YARN Resource Manager

This project provides a Swift wrapper of YARN Node Manager REST API:

- `YARNResourceManager()`: access to cluster information of YARN, including cluster and its metrics, scheduler, application submit, etc.

 
## Connect to YARN Resource Manager

To connect to your Hadoop YARN Resource Manager by Perfect, initialize a `YARNResourceManager()` object with sufficient parameters:

``` swift
// this connection could possibly do some basic operations
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088)
```
or connect to Hadoop YARN Node Manager with a valid user name:

``` swift
// add user name if need
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088, user: "your user name")
```

### Authentication
If using Kerberos to authenticate, please try codes below:

``` swift
// set auth to kerberos
let yarn = YARNResourceManager(host: "yarn.somehadoopdomain.com", port: 8088, user: "username", auth: .krb5)
```

### Parameters of YARNResourceManager Object
Item|Data Type|Description
----|---------|-----------
service|String|the service protocol of web request - http / https
host|String|the hostname or ip address of the Hadoop YARN Resource Manager
port|Int|the port of yarn host, default is 8088
auth|Authorization| .off or .krb5. Default value is .off
proxyUser|String|proxy user, if applicable
apibase|String|use this parameter *ONLY* the target server has a different api routine other than `/ws/v1/cluster`
timeout|Int|timeout in seconds, zero means never timeout during transfer

## Get General Information
Call `checkClusterInfo()` to get the general information of a YARN Resource Manager in form of a `ClusterInfo` structure:

``` swift
guard let i = try yarn.checkClusterInfo() else {
	print("unable to check cluster info")
	return
}//end guard
print(i.startedOn)
print(i.state)
print(i.hadoopVersion)
print(i.resourceManagerVersion)

```
Once called, `checkClusterInfo()` would return a `ClusterInfo` object, as described below:

### ClusterInfo Object

Item|Data Type|Description
----|---------|-----------
id|Int|The cluster id
startedOn|Int|The time the cluster started (in ms since epoch)
state|String|The ResourceManager state - valid values are: NOTINITED, INITED, STARTED, STOPPED
haState|String|The ResourceManager HA state - valid values are: INITIALIZING, ACTIVE, STANDBY, STOPPED
resourceManagerVersion|String|Version of the ResourceManager
resourceManagerBuildVersion|String|ResourceManager build string with build version, user, and checksum
resourceManagerVersionBuiltOn|String|Timestamp when ResourceManager was built (in ms since epoch)
hadoopVersion|String|Version of hadoop common
hadoopBuildVersion|String|Hadoop common build string with build version, user, and checksum
hadoopVersionBuiltOn|String|Timestamp when hadoop common was built(in ms since epoch)

## Cluster Metrics

Method `checkClusterMetrics()` returns a detailed info structure of cluster.

``` swift
guard let m = try yarn.checkClusterMetrics() else {
	// something wrong
}
print(m.availableMB)
print(m.availableVirtualCores)
print(m.allocatedVirtualCores)
print(m.totalMB)
```


Once called, `checkClusterMetrics()` would return a ClusterMetrics object as listed below:

### ClusterMetrics Object

Item|Data Type|Description
----|---------|-----------
appsSubmitted|Int|The number of applications submitted
appsCompleted|Int|The number of applications completed
appsPending|Int|The number of applications pending
appsRunning|Int|The number of applications running
appsFailed|Int|The number of applications failed
appsKilled|Int|The number of applications killed
reservedMB|Int|The amount of memory reserved in MB
availableMB|Int|The amount of memory available in MB
allocatedMB|Int|The amount of memory allocated in MB
totalMB|Int|The amount of total memory in MB
reservedVirtualCores|Int|The number of reserved virtual cores
availableVirtualCores|Int|The number of available virtual cores
allocatedVirtualCores|Int|The number of allocated virtual cores
totalVirtualCores|Int|The total number of virtual cores
containersAllocated|Int|The number of containers allocated
containersReserved|Int|The number of containers reserved
containersPending|Int|The number of containers pending
totalNodes|Int|The total number of nodes
activeNodes|Int|The number of active nodes
lostNodes|Int|The number of lost nodes
unhealthyNodes|Int|The number of unhealthy nodes
decommissionedNodes|Int|The number of nodes decommissioned
rebootedNodes|Int|The number of nodes rebooted

## Cluster Scheduler

Method `checkSchedulerInfo()` returns a detailed info structure of scheduler.

``` swift
guard let sch = try yarn.checkSchedulerInfo() else {
	// something wrong, must return
}
print(sch.capacity)
print(sch.maxCapacity)
print(sch.queueName)
print(sch.queues.count)
```

Once done, `checkSchedulerInfo()` would return a `SchedulerInfo` structure as listed below:

### SchedulerInfo Object

Item|Data Type|Description
----|---------|-----------
availNodeCapacity|Int|The available node capacity
capacity|Double|Configured queue capacity in percentage relative to its parent queue
maxCapacity|Double|Max capacity of the queue
maxQueueMemoryCapacity|Int|Configured maximum queue capacity in percentage relative to its parent queue
minQueueMemoryCapacity|Int|Minimum queue memory capacity
numContainers|Int|The number of containers
numNodes|Int|The total number of nodes
qstate|QState|State of the queue - valid values are: STOPPED, RUNNING
queueName|String|Name of the queue
queues| [Queue]|A collection of queue resources
rootQueue| FairQueue|A collection of root queue resources
totalNodeCapacity|Int|The total node capacity
type|String|Scheduler type - capacityScheduler
usedCapacity|Double|Used queue capacity in percentage
usedNodeCapacity|Int|The used node capacity

#### FairQueue Object
Item|Data Type|Description
----|---------|-----------
maxApps|Int|The maximum number of applications the queue can have
minResources|ResourcesUsed|The configured minimum resources that are guaranteed to the queue
maxResources|ResourcesUsed|The configured maximum resources that are allowed to the queue
usedResources|ResourcesUsed|The sum of resources allocated to containers within the queue
fairResources|ResourcesUsed|The queue’s fair share of resources
clusterResources|ResourcesUsed|The capacity of the cluster
queueName|String|The name of the queue
schedulingPolicy|String|The name of the scheduling policy used by the queue
childQueues|FairQueue|A collection of sub-queue information. Omitted if the queue has no childQueues.
type|String|type of the queue - fairSchedulerLeafQueueInfo
numActiveApps|Int|The number of active applications in this queue
numPendingApps|Int|The number of pending applications in this queue

#### Queue Object

Item|Data Type|Description
----|---------|-----------
absoluteCapacity|Double|Absolute capacity percentage this queue can use of entire cluster
absoluteMaxCapacity|Double|Absolute maximum capacity percentage this queue can use of the entire cluster
absoluteUsedCapacity|Double|Absolute used capacity percentage this queue is using of the entire cluster
capacity|Double|Configured queue capacity in percentage relative to its parent queue
maxActiveApplications|Int|The maximum number of active applications this queue can have
maxActiveApplicationsPerUser|Int|The maximum number of active applications per user this queue can have
maxApplications|Int|The maximum number of applications this queue can have
maxApplicationsPerUser|Int|The maximum number of applications per user this queue can have
maxCapacity|Double|Configured maximum queue capacity in percentage relative to its parent queue
numActiveApplications|Int|The number of active applications in this queue
numApplications|Int|The number of applications currently in the queue
numContainers|Int|The number of containers being used
numPendingApplications|Int|The number of pending applications in this queue
queueName|String|The name of the queue
queues|[Queue]|A collection of sub-queue information. Omitted if the queue has no sub-queues.
resourcesUsed|ResourcesUsed|The total amount of resources used by this queue
state|String|The state of the queue
type|String|type of the queue - capacitySchedulerLeafQueueInfo
usedCapacity|Double|Used queue capacity in percentage
usedResources|String|A string describing the current resources used by the queue
userLimit|Int|The minimum user limit percent set in the configuration
userLimitFactor|Double|The user limit factor set in the configuration
users|[User]|A collection of user objects containing resources used, see below:

#### User Object

Item|Data Type|Description
----|---------|-----------
username|String|The username of the user using the resources
resourcesUsed|ResourcesUsed |The amount of resources used by the user in this queue, see definition below
numActiveApplications|Int|The number of active applications for this user in this queue
numPendingApplications|Int|The number of pending applications for this user in this queue

#### ResourcesUsed Object

Item|Data Type|Description
----|---------|-----------
memory|int|Memory required for each container
vCores|int|Virtual cores required for each container

## Cluster Nodes

### Check All Nodes

Method `checkClusterNodes()` returns an array of node info of cluster.

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

Once done, the `checkClusterNodes()` would return an array of `Node` object, as described below:

#### Node Object

Item|Data Type|Description
----|---------|-----------
rack|String|The rack location of this node
state|String|State of the node - valid values are: NEW, RUNNING, UNHEALTHY, DECOMMISSIONED, LOST, REBOOTED
id|String|The node id
nodeHostName|String|The host name of the node
nodeHTTPAddress|String|The nodes HTTP address
healthStatus|String|The health status of the node - Healthy or Unhealthy
healthReport|String|A detailed health report
lastHealthUpdate|Int|The last time the node reported its health (in ms since epoch)
usedMemoryMB|Int|The total amount of memory currently used on the node (in MB)
availMemoryMB|Int|The total amount of memory currently available on the node (in MB)
usedVirtualCores|Int|The total number of vCores currently used on the node
availableVirtualCores|Int|The total number of vCores available on the node
numContainers|int|The total number of containers currently running on the node

### Check A Node

Method `checkClusterNode()` returns a detailed info structure of a node.

``` swift
guard let n = try yarn.checkClusterNode(id: "host.domain.com:8041") else {
	// something wrong, must return
}
```

## Applications on Cluster

### Check All Applications

Method `checkApps()` returns an array of APP structure.
``` swift
let apps = try yarn.checkApps()
// or alternatively, you can filter out those APPs you want by setting query parameters:
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

Once done, the `checkApps()` would return an array of APP objects, as described below:

#### APP Object

Item|Data Type|Description
----|---------|-----------
id|String|The application id
user|String|The user who started the application
name|String|The application name
applicationType|String|The application type
queue|String|The queue the application was submitted to
state|String|The application state according to the ResourceManager - valid values are members of the YarnApplicationState enum: NEW, NEW_SAVING, SUBMITTED, ACCEPTED, RUNNING, FINISHED, FAILED, KILLED
finalStatus|String|The final status of the application if finished - reported by the application itself - valid values are: UNDEFINED, SUCCEEDED, FAILED, KILLED
progress|Double|The progress of the application as a percent
trackingUI|String|Where the tracking url is currently pointing - History (for history server) or ApplicationMaster
trackingUrl|String|The web URL that can be used to track the application
diagnostics|String|Detailed diagnostics information
clusterId|Int|The cluster id
startedTime|Int|The time in which application started (in ms since epoch)
finishedTime|Int|The time in which the application finished (in ms since epoch)
elapsedTime|Int|The elapsed time since the application started (in ms)
amContainerLogs|String|The URL of the application master container logs
amHostHttpAddress|String|The nodes http address of the application master
amRPCAddress|String|The RPC address of the application master
allocatedMB|Int|The sum of memory in MB allocated to the application’s running containers
allocatedVCores|Int|The sum of virtual cores allocated to the application’s running containers
runningContainers|Int|The number of containers currently running for the application
memorySeconds|Int|The amount of memory the application has allocated (megabyte-seconds)
vcoreSeconds|Int|The amount of CPU resources the application has allocated (virtual core-seconds)
unmanagedApplication|Bool|Is the application unmanaged.
applicationPriority|Int|priority of the submitted application
appNodeLabelExpression|String|Node Label expression which is used to identify the nodes on which application’s containers are expected to run by default.
amNodeLabelExpression|String|Node Label expression which is used to identify the node on which application’s AM container is expected to run.

### Check A Specific Application

Method `checkApp()` returns a specific APP Object.

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
The return value is an APP structure, please check the above APP object for detail.

### Apply A New Application

Method `newApplication()` returns a new application handler.

``` swift
guard let a = try yarn.newApplication() else {
	// cannot create a new application, must return
}
```

Once completed, the `newApplication()` method will return a NewApplication object, as described below:

#### NewApplication Object

Item|Data Type|Description
----|---------|-----------
id|String|The newly created application id
maximumResourceCapability|ResourcesUsed|The maximum resource capabilities available on this cluster

#### ResourcesUsed Object

The `NewApplication` object will contain a special object called `ResourceUsed`, as described below:

Item|Data Type|Description
----|---------|-----------
memory|Int|The maximum memory available for a container
vCores|Int|The maximum number of cores available for a container


### Submit Modification to An Application

Method `submit()` can submit modifications to a specific application.

``` swift
// create an empty application to fill in the blanks
let sum = SubmitApplication()

// *MUST* set the application id
sum.id =  "application_1484231633049_0025"
// set the application name
sum.name = "test"

// allocate a blank local resource
let local = LocalResource(resource: "hdfs://localhost:9000/user/rockywei/DistributedShell/demo-app/AppMaster.jar", type: .FILE, visibility: .APPLICATION, size: 43004, timestamp: 1405452071209)

// assign the resource into an array
let localResources = Entries([Entry(key:"AppMaster.jar", value: local)])

// *MUST* fill in the field of map reduce command
let commands = Commands("/hdp/bin/hadoop jar /hdp/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha1.jar grep input output 'dfs[a-z.]+'")

// setup environment as need
let environments = Entries([Entry(key:"DISTRIBUTEDSHELLSCRIPTTIMESTAMP", value: "1405459400754"), Entry(key:"CLASSPATH", value:"{{CLASSPATH}}<CPS>./*<CPS>{{HADOOP_CONF_DIR}}<CPS>{{HADOOP_COMMON_HOME}}/share/hadoop/common/*<CPS>{{HADOOP_COMMON_HOME}}/share/hadoop/common/lib/*<CPS>{{HADOOP_HDFS_HOME}}/share/hadoop/hdfs/*<CPS>{{HADOOP_HDFS_HOME}}/share/hadoop/hdfs/lib/*<CPS>{{HADOOP_YARN_HOME}}/share/hadoop/yarn/*<CPS>{{HADOOP_YARN_HOME}}/share/hadoop/yarn/lib/*<CPS>./log4j.properties"), Entry(key:"DISTRIBUTEDSHELLSCRIPTLEN", value:6), Entry(key:"DISTRIBUTEDSHELLSCRIPTLOCATION", value: "hdfs://localhost:9000/user/rockywei/demo-app/shellCommands")])

// specify the container info
sum.amContainerSpec = AmContainerSpec(localResources: localResources, environment: environments, commands: commands)

// set other information as need
sum.unmanagedAM = false
sum.maxAppAttempts = 2
sum.resource = ResourceRequest(memory: 1024, vCores: 1)
sum.type = "MapReduce"
sum.keepContainersAcrossApplicationAttempts = false

// set the log context
sum.logAggregationContext = LogAggregationContext(logIncludePattern: "file1", logExcludePattern: "file2", rolledLogIncludePattern: "file3", rolledLogExcludePattern: "file4", logAggregationPolicyClassName: "org.apache.hadoop.yarn.server.nodemanager.containermanager.logaggregation.AllContainerLogAggregationPolicy", logAggregationPolicyParameters: "")

// set the failures validity interval
sum.attemptFailuresValidityInterval = 3600000

// set the reservationId, if possible
sum.reservationId = "reservation_1454114874_1"
sum.amBlackListingRequests = AmBlackListingRequests(amBlackListingEnabled: true, disableFailureThreshold: 0.01)

try yarn.submit(application: sum)
```

### SubmitApplication Object

`SubmitApplication` class contains quite a few sub types, as described below:

Item|Data Type|Description
----|---------|-----------
id|String|The application id
name|String|The application name
queue|String|The name of the queue to which the application should be submitted
priority|Int|The priority of the application
amContainerSpec|AmContainerSpec|The application master container launch context, described below
unmanagedAM|Bool|Is the application using an unmanaged application master
maxAppAttempts|int|The max number of attempts for this application
resource|ResourceRequest|The resources the application master requires, described below
type|String|The application type(MapReduce, Pig, Hive, etc)
keepContainersAcrossApplicationAttempts|Bool|Should YARN keep the containers used by this application instead of destroying them
tags|[String]|List of application tags, please see the request examples on how to specify the tags
logAggregationContext|LogAggregationContext|Represents all of the information needed by the NodeManager to handle the logs for this application
attemptFailuresValidityInterval|Int|The failure number will no take attempt failures which happen out of the validityInterval into failure count
reservationId|string|Represent the unique id of the corresponding reserved resource allocation in the scheduler
amBlackListingRequests| AmBlackListingRequests |Contains blacklisting information such as “enable/disable AM blacklisting” and “disable failure threshold”


#### AmContainerSpec Object

Item|Data Type|Description
----|---------|-----------
localResources|Entries|Object describing the resources that need to be localized, described below
environment|Entries|Environment variables for your containers, specified as key value pairs
commands|Commands|The commands for launching your container, in the order in which they should be executed
serviceData|Entries|Application specific service data; key is the name of the auxiliary service, value is base-64 encoding of the data you wish to pass
credentials|Credentials|The credentials required for your application to run, described below
applicationAcls| Entries |ACLs for your application; the key can be “VIEW_APP” or “MODIFY_APP”, the value is the list of users with the permissions

#### LocalResource Object

Item|Data Type|Description
----|---------|-----------
resource|String|Location of the resource to be localized
type|ResourceType|Type of the resource; options are “ARCHIVE”, “FILE”, and “PATTERN”
visibility|Visibility|Visibility the resource to be localized; options are “PUBLIC”, “PRIVATE”, and “APPLICATION”
size|Int|Size of the resource to be localized
timestamp|Int|Timestamp of the resource to be localized

#### Commands Object

Currently Commands object only one string field named `command`.
⚠️**NOTE**⚠️ According to Hadoop 3.0 alpha document, commands should be an array of command strings with a sequential meaning, however, the example given in Hadoop 3.0 alpha indicates only one command. This feature is experimental and subject to change in future.

#### Credentials Object

Item|Data Type|Description
----|---------|-----------
tokens|[String:String]|Tokens that you wish to pass to your application, specified as key-value pairs. The key is an identifier for the token and the value is the token(which should be obtained using the respective web-services)
secrets|[String:String]|Secrets that you wish to use in your application, specified as key-value pairs. They key is an identifier and the value is the base-64 encoding of the secret

#### ResourceRequest Object

Item|Data Type|Description
----|---------|-----------
memory|int|Memory required for each container
vCores|int|Virtual cores required for each container

#### Entries Object

The `Entries` Object has only one field of element: `entry`, which is an array of `Entry` object or `EncryptedEntry` object. Each `Entry` has two elements: a string `key` and a `Any` type `value`.

The difference between `Entry` and `EncryptedEntry` is that the value of `EncryptedEntry` will be encoded in base64 format before submission, providing the type of value is either `String` or `[UInt8]`.

#### AmBlackListingRequests Object

Item|Data Type|Description
----|---------|-----------
amBlackListingEnabled|Bool|Whether AM Blacklisting is enabled
disableFailureThreshold|Float|AM Blacklisting disable failure threshold

#### LogAggregationContext Object

Item|Data Type|Description
----|---------|-----------
logIncludePattern|String|The log files which match the defined include pattern will be uploaded when the application finishes
logExcludePattern|String|The log files which match the defined exclude pattern will not be uploaded when the application finishes
rolledLogIncludePattern|String|The log files which match the defined include pattern will be aggregated in a rolling fashion
rolledLogExcludePattern|String|The log files which match the defined exclude pattern will not be aggregated in a rolling fashion
logAggregationPolicyClassName|String|The policy which will be used by NodeManager to aggregate the logs
logAggregationPolicyParameters|String|The parameters passed to the policy class

## Application State Control

Method `getApplicationStatus()` returns the current state of application.

``` swift
guard let state = try yarn.getApplicationStatus(id:  "application_1484231633049_0025") else {
	// something wrong, must return
}
print(state)
```

Method `setApplicationStatus()` can set the current state of application to a designated one.

``` swift
try yarn.setApplicationStatus(id:  "application_1484231633049_0025", state: .KILLED)
```

Valid States includes:

``` swift
NEW, NEW_SAVING, SUBMITTED, ACCEPTED, RUNNING, FINISHED, FAILED, KILLED
```

## Application Queue Control

Method `getApplicationQueue()` returns the current queue name of application.

``` swift
guard let queue = try yarn.getApplicationQueue(id:  "application_1484231633049_0025") else {
	// something wrong, must return
}
print(queue)
```

Method `setApplicationQueue()` can set the current queue name of application to a designated one.

``` swift
try yarn.setApplicationQueue(id:  "application_1484231633049_0025", queue:"a1a")
```

## Application Priority Control

Method `getApplicationPriority()` returns the current priority of application.

``` swift
guard let priority = try yarn.getApplicationPriority(id:  "application_1484231633049_0025") else {
	// something wrong, must return
}
print(priority)
```

Currently the returned priority is an integer such as `0`.

Method `setApplicationPriority()` can set the current priority of application to a designated one.

``` swift
try yarn.setApplicationPriority(id:  "application_1484231633049_0025", priority: 1)
```

## Application Attempts

Method `checkAppAttempts()` returns an array of Attempt object of application.

``` swift
guard let attempts = try yarn.checkAppAttempts(id:  "application_1484231633049_0025") else {
	// something wrong, must return
}
attempts.forEach { attempt in
	print(attempt.containerId)
	print(attempt.id)
	print(attempt.nodeHttpAddress)
	print(attempt.nodeId)
	print(attempt.startTime)
}//next
```

Once done, `checkAppAttempts()` would return an array of AppAttempt object, as described below:

### AppAttempt Object

Item|Data Type|Description
----|---------|-----------
id|String|The app attempt id
nodeId|String|The node id of the node the attempt ran on
nodeHttpAddress|String|The node http address of the node the attempt ran on
logsLink|String|The http link to the app attempt logs
containerId|String|The id of the container for the app attempt
startTime|Int|The start time of the attempt (in ms since epoch)

## Application Statistics

Method `checkAppStatistics()` returns an array of statistic variables of application.
``` swift
let sta = try yarn.checkAppStatistics(states: [APP.State.FINISHED, APP.State.RUNNING])
sta.forEach{ s in
	print(s.count)
	print(s.state)
	print(s.type)
}//next s
```

`checkAppStatistics()` allows user to perform a query with two additional parameters:

Parameter|Data Type|Description
---------|---------|-----------
states|[APP.State]|states of the applications. If states is not provided, the API will enumerate all application states and return the counts of them.
applicationTypes|[String]|types of the applications. If applicationTypes is not provided, the API will count the applications of any application type. In this case, the response shows * to indicate any application type. Note that we only support at most one applicationType temporarily, such as `applicationTypes: ["MapReduce"]`
