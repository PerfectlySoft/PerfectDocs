# PerfectHadoop

该项目实现了一个对 WebHDFS 网络接口的封装，用于访问 Hadoop 服务器。

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分。本软件包可独立使用，因此使用时可以脱离PerfectLib等其他组件。

请确保您已经安装并激活了最新版本的 Swift 4.0 tool chain 工具链。

### 问题报告、内容贡献和客户支持

我们目前正在过渡到使用JIRA来处理所有源代码资源合并申请、修复漏洞以及其它有关问题。因此，GitHub 的“issues”问题报告功能已经被禁用了。

如果您发现了问题，或者希望为改进本文提供意见和建议，[请在这里指出](http://jira.perfect.org:8080/servicedesk/customer/portal/1).

在您开始之前，请参阅[目前待解决的问题清单](http://jira.perfect.org:8080/projects/ISS/issues).

## 版本兼容性
PerfectHadoop 目前支持 Hadoop 3.0.0，以及 2.7.3 的部分功能。

## 编译
请在您的 Package.swift 文件中增加以下内容

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Hadoop.git", majorVersion: 3)
```

并在您的源程序部分增加以下函数库声明：
``` swift
import PerfectHadoop
```
## 错误处理 - `Exception`

由于基于REST API，大多数 Perfect-Hadoop 库函数在出错时会抛出一个`Exception`对象，用户可以捕捉该错误并检查三元组`(url, header, body)`如下列程序所示：

``` swift
do {
	// 执行任何一个 Perfect Hadoop 操作，包括 WebHDFS / MapReduce / YARN，所有的操作
	...
}
catch(Exception.unexpectedResponse(let (url, header, body))) {
	print("出现REST API异常： \(url)\n\(header)\n\(body)")
}
catch (let err){
	print("其它错误：\(err)")
}
```

## 用户手册
- WebHDFS: [Perfect-HDFS](HadoopWebHDFS.md)
- MapReduce: 
	* [Perfect-MapReduce 应用程序接口 API](HadoopMapReduceMaster.md)  ⚠️ 试验版本 ⚠️ 
	* [Perfect-MapReduce 历史服务器接口 API](HadoopMapReduceHistory.md)
- YARN:
	* [Perfect-YARN 节点管理器](HadoopYARNNodeManager.md)
	* [Perfect-YARN 资源管理器](HadoopYARNResourceManager.md)



## 更多信息
关于本项目更多内容，请参考[perfect.org](http://perfect.org).
