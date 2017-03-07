# PerfectHadoop: YARN Node Manager

This project provides a Swift wrapper of YARN Node Manager REST API:

- `YARNNodeManager()`: access to all Applications and Containers information of a YARN manager

 
## Connect to YARN Node Manager

To connect to your Hadoop YARN Node Manager by Perfect, initialize a `YARNNodeManager()` object with sufficient parameters:

``` swift
// this connection could possibly do some basic operations
let nm = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042)
```
or connect to Hadoop YARN Node Manager with a valid user name:

``` swift
// add user name if need
let nm = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042, user: "your user name")
```

### Authentication
If using Kerberos to authenticate, please try codes below:

``` swift
// set auth to kerberos
let yarn = YARNNodeManager(host: "yarnNodeManager.somehadoopdomain.com", port: 8042, user: "username", auth: .krb5)
```

### Parameters of YARNNodeManager Object
Item|Data Type|Description
----|---------|-----------
service|String|the service protocol of web request - http / https
host|String|the hostname or ip address of the Hadoop YARN node manager
port|Int|the port of yarn host, default is 8042
auth|Authorization| .off or .krb5. Default value is .off
proxyUser|String|proxy user, if applicable
apibase|String|use this parameter *ONLY* the target server has a different api routine other than `/ws/v1/node`
timeout|Int|timeout in seconds, zero means never timeout during transfer

## Get General Information
Call `checkOverall()` to get the general information of a YARN node manager in form of a `YARNNodeManager.Info` structure:

``` swift
guard let inf = try yarn.checkOverall() else {
	// something goes wrong here
}
print(inf.id)
print(inf.nodeHostName)
print(inf.healthReport)
```

### Members of `YARNNodeManager.NodeInfo`

Item|Data Type|Description
----|---------|-----------
id|String|The NodeManager id
nodeHostName|String|The host name of the NodeManager
totalPmemAllocatedContainersMB|Int|The amount of physical memory allocated for use by containers in MB
totalVmemAllocatedContainersMB|Int|The amount of virtual memory allocated for use by containers in MB
totalVCoresAllocatedContainers|Int|The number of virtual cores allocated for use by containers
lastNodeUpdateTime|Int|The last timestamp at which the health report was received (in ms since epoch)
healthReport|String|The diagnostic health report of the node
nodeHealthy|Bool|true/false indicator of if the node is healthy
nodeManagerVersion|String|Version of the NodeManager
nodeManagerBuildVersion|String|NodeManager build string with build version, user, and checksum
nodeManagerVersionBuiltOn|String|Timestamp when NodeManager was built(in ms since epoch)
hadoopVersion|String|Version of hadoop common
hadoopBuildVersion|String|Hadoop common build string with build version, user, and checksum
hadoopVersionBuiltOn|String|Timestamp when hadoop common was built(in ms since epoch)

## Applications

Call `checkApps()` to return an array of `APP` structure.
With the Applications API, you can obtain a collection of resources, each of which represents an application.

``` swift
let apps = try yarn.checkApps()
apps.forEach { a in
  print(" ------------  applications on node -----------")
  print(a.id)
  print(a.containerids)
  print(a.state)
}//next
```

### Data Structure of APP
Item|Data Type|Description
----|---------|-----------
id|String|The application id
user|String|The user who started the application
state|APP.State|The state of the application - valid states are: NEW, INITING, RUNNING, FINISHING_CONTAINERS_WAIT, APPLICATION_RESOURCES_CLEANINGUP, FINISHED
containerids|[String]]|The list of containerids currently being used by the application on this node. If not present then no containers are currently running for this application.

## Check A Specific Application

If an application id is available, you can call `checkApp()` to query information on nodes:

``` swift
guard let app = try yarn.checkApp("application_1326121700862_0005") else {
	// something wrong
	print("check app failed")
	return
}
// print out all container ids of the application
print(app.containerids)
```
`checkApp()` will return an `APP` object, as described above.

## Check Containers
Call `checkContainers()` to check all containers on the current node:

``` swift
// get all containers
let containers = try ynode.checkContainers()

// check how many containers on node
print(containers.count)

containers.forEach { container in
	print(container.id)
	print(container.state)
	print(container.nodeId)
	print(container.diagnostics)
	/// ...
}

```
`checkContainers()` returns an array of `Container` structure, as described below:

### Container Object
Item|Data Type|Description
----|---------|-----------
id|String|The container id
state|String|State of the container - valid states are: NEW, LOCALIZING, LOCALIZATION_FAILED, LOCALIZED, RUNNING, EXITED_WITH_SUCCESS, EXITED_WITH_FAILURE, KILLING, CONTAINER_CLEANEDUP_AFTER_KILL, CONTAINER_RESOURCES_CLEANINGUP, DONE
nodeId|String|The id of the node the container is on
containerLogsLink|String|The http link to the container logs
user|String|The user name of the user which started the container
exitCode|Int|Exit code of the container
diagnostics|String|A diagnostic message for failed containers
totalMemoryNeededMB|Int|Total amout of memory needed by the container (in MB)
totalVCoresNeeded|Int|Total number of virtual cores needed by the container

## Check a Specific Container

If a container id is available, you can also use `checkContainer()` to check a specific container:

``` swift
guard let container = try ynode.checkContainer(id: container.id) else {
	// checking container failed
}
print(container.id)
print(container.state)
print(container.nodeId)
print(container.diagnostics)
```

This method will return a `Container` object as described above.
