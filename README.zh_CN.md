# Perfect 文档库 [English](https://github.com/PerfectlySoft/PerfectDocs)
<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>
    <a href="https://gitter.im/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_2_Git.jpg" alt="Chat on Gitter" />
    </a>
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat" alt="Swift 3.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="https://gitter.im/PerfectlySoft/Perfect?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge" target="_blank">
        <img src="https://img.shields.io/badge/Gitter-Join%20Chat-brightgreen.svg" alt="Join the chat at https://gitter.im/PerfectlySoft/Perfect">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>

本文献包括所有您需要运行和使用Perfec的软件参考手册和接口函数有关内容。

### 问题报告、内容贡献和客户支持

我们目前正在过渡到使用JIRA来处理所有源代码资源合并申请、修复漏洞以及其它有关问题。因此，GitHub 的“issues”问题报告功能已经被禁用了。

如果您发现了问题，或者希望为改进本文提供意见和建议，[请在这里指出](http://jira.perfect.org:8080/servicedesk/customer/portal/1).

在您开始之前，请参阅[目前待解决的问题清单](http://jira.perfect.org:8080/projects/ISS/issues).

### Contributing

我们欢迎所有贡献以及对Perfect文档提高的宝贵意见。我们欢迎您为Perfect付出宝贵的支持。如果您发现了任何文字或者内容有错误，或者有任何建议，请[提交一个代码上传请求，或在JIRA上报告问题](http://jira.perfect.org:8080/servicedesk/customer/portal/1/user/login?destination=portal%2F1).

我们由一个专用[Perfect文档生成器](https://github.com/PerfectlySoft/PerfectDocGenerator)，也是由Swift语言编写，能够将本资源库目录下的MD文档转换为HTML网页。该系统将很快启动。

## 目录

* [简介](guide.zh_CN/introduction.md)
* [快速上手](guide.zh_CN/gettingStarted.md)
* [从模板项目开始](guide.zh_CN/gettingStartedFromScratch.md)
* [HTTP和Web服务基础](guide.zh_CN/WebServicesPrimer.md)
* [代码资源库结构](guide.zh_CN/repositoryLayout.md)
* [用SPM软件包管理器编译项目](guide.zh_CN/buildingWithSPM.md)
* [处理HTTP请求](guide.zh_CN/handlingRequests.md)
	* [HTTP路由](guide.zh_CN/routing.md)
	* [HTTPRequest请求对象](guide.zh_CN/HTTPRequest.md)
	 	* [使用表单](guide.zh_CN/formData.md)
		* [文件上传](guide.zh_CN/fileUploads.md)
	* [HTTPResponse响应对象](guide.zh_CN/HTTPResponse.md)
	* [HTTP请求与响应过滤器](guide.zh_CN/filters.md)
	* [JSON数据转换](guide.zh_CN/JSON.md)
	* [静态文件](guide.zh_CN/staticFileContent.md)
	* [Mustache页面模板](guide.zh_CN/mustache.md)
* [基本工具](guide.zh_CN/utilities.md)
	* [字节流转换](guide.zh_CN/bytes.md)
	* [文件操作](guide.zh_CN/file.md)
	* [目录与路径](guide.zh_CN/dir.md)
	* [线程](guide.zh_CN/thread.md)
	* [网络](guide.zh_CN/net.md) @kjessup
	* [UUID唯一标识符](guide.zh_CN/UUID.md)
	* [SysProcess系统进程](guide.zh_CN/sysProcess.md)
	* [日志](guide.zh_CN/log.md)
	* [CURL联网传输](guide.zh_CN/cURL.md)
	* [XML数据转换](guide.zh_CN/xml.md) @kjessup
	* [Zip压缩](guide.zh_CN/zip.md)
* [数据库连接器](guide.zh_CN/databaseConnectors.md)
	* [SQLite](guide.zh_CN/SQLite.md)
	* [MySQL](guide.zh_CN/MySQL.md)
	* [PostgreSQL](guide.zh_CN/PostgreSQL.md)
	* [MongoDB](guide.zh_CN/MongoDB.md)
		* [MongoDB 数据库](guide.zh_CN/MongoDB-Database.md)
		* [MongoDB 集合](guide.zh_CN/MongoDB-Collections.md)
		* [MongoDB 客户端](guide.zh_CN/MongoDB-Client.md)
		* [BSON 数据转换](guide.zh_CN/MongoDB-BSON.md)
	* [Redis](guide.zh_CN/Redis.md) @ kjessup
	* [Filemaker](guide.zh_CN/filemaker.md)
* [WebSockets](guide.zh_CN/webSockets.md) @kjessup
* [iOS 消息与通知](guide.zh_CN/iOSNotifications.md) @kjessup
* [发行与部署](guide.zh_CN/deployment.md)
	* [Ubuntu 15.10](guide.zh_CN/deployment-Ubuntu1510.md)
	* Docker
	* Heroku
	* Azure
	* AWS
	* Linode
	* [Digital Ocean](guide.zh_CN/deployment-DigitalOcean.md)
* 平台安装说明
	* [Ubuntu 16.04 系统服务安装指南](guide.zh_CN/starting-services.md)
