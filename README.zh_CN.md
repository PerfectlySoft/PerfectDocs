# Perfect 文档库 [English](https://github.com/PerfectlySoft/PerfectDocs)

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involved with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="http://stackoverflow.com/questions/tagged/perfect" target="_blank">
        <img src="http://www.perfect.org/github/perfect_gh_button_2_SO.jpg" alt="Stack Overflow" />
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
        <img src="https://img.shields.io/badge/Swift-4.1-orange.svg?style=flat" alt="Swift 4.1">
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

* [简介](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/introduction.md)
* [快速上手](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/gettingStarted.md)
* [从模板项目开始](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/gettingStartedFromScratch.md)
* [HTTP和Web服务基础](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/WebServicesPrimer.md)
* [代码资源库结构](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/repositoryLayout.md)
* [用SPM软件包管理器编译项目](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/buildingWithSPM.md)
* [HTTP服务器配置与启动](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HTTPServer.md)
* [处理HTTP请求](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/handlingRequests.md)
	* [HTTP路由](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/routing.md)
	* [HTTPRequest请求对象](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HTTPRequest.md)
		* [使用表单](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/formData.md)
		* [文件上传](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/fileUploads.md)
	* [HTTPResponse响应对象](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HTTPResponse.md)
	* [HTTP请求与响应过滤器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/filters.md)
		* [Web 重定向过滤器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/webRedirects.md)
	* [Session 会话管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/sessions.md)
		* [CSRF 安全功能](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/csrf.md)
		* [CORS 安全功能](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/cors.md)
	* [用户身份认证](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/authentication.md)	
	* [GSS-SPNEGO 安全插件](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/SPNEGO.md)
	* [JSON数据转换](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/JSON.md)
	* [静态文件](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/staticFileContent.md)
	* [Mustache页面模板](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/mustache.md)
	* [Markdown](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Markdown.md)
	* [HTTP请求日志记录器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HTTPRequestLogging.md)
	* [日志文件格式](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/logFiles.md)
	* [远程日志服务器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/logRemote.md)
* [CRUD](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/crud.md)
* [WebSockets](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/webSockets.md)
* [基本工具](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/utilities.md)
	* [字节流转换](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/bytes.md)
	* [文件操作](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/file.md)
	* [目录与路径](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/dir.md)
	* [环境变量](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/env.md)
	* [线程](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/thread.md)
	* [网络](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/net.md) @kjessup
	* [OAuth2 身份授权](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/OAuth2.md)
	* [UUID唯一标识符](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/UUID.md)
	* [SysProcess系统进程](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/sysProcess.md)
	* [日志](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/log.md)
	* [日志文件](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/logFiles.md)
	* [CURL联网传输](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/cURL.md)
	* [XML数据转换](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/xml.md)
	* [INI配置](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/ini.md)
	* [Zip压缩](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/zip.md)
	* [Crypto 加密](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/crypto.md)
	* [SMTP 简单邮件协议](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/SMTP.md)
	* [谷歌网站分析协议l](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/GoogleAnalytics.md)
	* [定时器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/repeater.md)
* [数据库连接器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/databaseConnectors.md)
	* [SQLite](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/SQLite.md)
	* [MySQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MySQL.md)
	* [MariaDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MariaDB.md)
	* [PostgreSQL](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/PostgreSQL.md)
	* [MongoDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB.md)
		* [MongoDB 数据库](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB-Database.md)
		* [MongoDB 集合](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB-Collections.md)
		* [MongoDB 客户端](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB-Client.md)
		* [BSON 数据转换](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB-BSON.md)
		* [GridFS 文件系统](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/MongoDB-GridFS.md)
	* [CouchDB](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/CouchDB.md)
	* [LDAP](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/LDAP.md)
	* [Redis](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Redis.md) @kjessup
	* [FileMaker](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/filemaker.md)
* [iOS 消息与通知](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/iOSNotifications.md) @kjessup
* [发行与部署](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/deployment.md)
	* [Ubuntu](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/deployment-Ubuntu.md)
	* Docker
	* Heroku
	* Azure
	* AWS
	* Linode
	* [Digital Ocean](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/deployment-DigitalOcean.md)

* 服务器性能远程监控
	* [SysInfo](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/SYSINFO.md)
	* [New Relic](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/NEWRELIC.md)

* 消息队列及服务器集群
	* [Kafka](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Kafka.md)
	* [Mosquitto](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/mosquitto.md)
	* [ZooKeeper (Linux Only)](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/ZooKeeper.md)

* 大数据、人工智能及机器学习
	* [TensorFlow](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/tensorflow.md)
	* [Hadoop](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Hadoop.md)
		* [HDFS](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/HadoopWebHDFS.md)
		* [MapReduce Master 应用程序服务器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN//HadoopMapReduceMaster.md)
		* [MapReduce 历史服务器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN//HadoopMapReduceHistory.md)
		* [YARN 节点管理器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN//HadoopYARNNodeManager.md)
		* [YARN 资源管理器](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN//HadoopYARNResourceManager.md)

* 平台安装说明
	* [Ubuntu 16.04 系统服务安装指南](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/starting-services.md)

* 语言扩展：
	* [Python](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/python.md)

## StORM：Swift的对象关系管理（ORM）

StORM 并非 Perfect.org 发行的项目，但是 Perfect 的函数库将 StORM 集成到了整个框架范围内，因此本项目与StORM存在共同的作者。

* [StORM：Swift的对象关系管理（ORM） ](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM.md)
	* [StORM简介](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM.md)
	* [设置对象](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Setting-up-a-class.md)
	* [数据记录行操作：存、取和删除](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Saving-Retrieving-and-Deleting-Rows.md)
	* [StORM游标](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Cursor.md)
	* [插入数据](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Insert.md)
	* [更新数据](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-Update.md)
	* [StORM 全局全周期事件](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORMLifecycleEvents.md)
	* [PostgresStORM 对象关系管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-PostgreSQL.md)
	* [SQLiteStORM 对象关系管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-SQLite.md)
	* [MySQLStORM 对象关系管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-MySQL.md)
	* [Apache CouchDB 对象关系管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-CouchDB.md)
	* [MongoDB 对象关系管理](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM-MongoDB.md)

 ## Perfect Turnstile – 用户身份验证管理

 [Turnstile](https://github.com/stormpath/Turnstile) 是一个用于用户身份跨平台认证的开源软件框架体系 [Stormpath](https://github.com/stormpath) 。感谢 [Edward Jiang](https://github.com/edjiang) 为 Perfect 提供了 Turnstile。

**⚠️注意⚠️** Turnstile 作者已经终止维护该项目，详见 [Storm API 移植常见问题](https://stormpath.com/oktaplusstormpath?utm_source=github&utm_medium=readme&utm-campaign=okta-announcement)。

* [Perfect-Turnstile](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Turnstile.md)
	* [Perfect Turnstile 与 SQLite 集成](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Turnstile.md)
	* [Perfect Turnstile 与 PostgreSQL 集成](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Turnstile.md)
	* [Perfect Turnstile 与 MySQL 集成](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Turnstile.md)
	* [Perfect Turnstile 与 CouchDB 集成](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/Turnstile.md)
