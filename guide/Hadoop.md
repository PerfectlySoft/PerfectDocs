# PerfectHadoop 

This project provides a set of Swift classes which enable access to Hadoop servers.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project. It was written to be stand-alone and so does not require PerfectLib or any other components.

Ensure you have installed and activated the latest Swift 4.0 tool chain.

## Release Note
PerfectHadoop supports Hadoop 3.0.0 with a limitation on 2.7.3.

## Building
Add this project as a dependency in your Package.swift file.

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Hadoop.git", majorVersion: 3)
```

Then please add the following line to the beginning part of swift sources:
``` swift
import PerfectHadoop
```

## Error Handle - `Exception`

In case of operation failure, an exception might be thrown out. In most cases of Perfect-Hadoop, the library would probably throw a `Exception` object. User can catch it and check a tuple `(url, header, body)` of the failure, as demo below:

``` swift
do {
	// some Perfect Hadoop operations, including WebHDFS / MapReduce / YARN, all of them:
	...
}
catch(Exception.unexpectedResponse(let (url, header, body))) {
	print("Exception: \(url)\n\(header)\n\(body)")
}
catch (let err){
	print("Other Error:\(err)")
}
```

## User Manual
- WebHDFS: [Perfect-HDFS](HadoopWebHDFS.md)
- MapReduce: 
	* [Perfect-MapReduce Application Master API](HadoopMapReduceMaster.md) ⚠️ Experimental  ⚠️
	* [Perfect-MapReduce History Server API](HadoopMapReduceHistory.md)
- YARN:
	* [Perfect-YARN Node Manager](HadoopYARNNodeManager.md)
	* [Perfect-YARN Resource Manager](HadoopYARNResourceManager.md)


