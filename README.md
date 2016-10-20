# Perfect Documentation Library [简体中文](README.zh_CN.md)

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
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
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>

This library contains all the reference documentation and API reference-related material you need to run and use Perfect.

### Issues, Pull Requests, and Support

We have transitioned to using JIRA for dealing with all pull requests, bugs, and any other support-related issues. Therefore, the GitHub "issues" tab has been disabled.

If you find a bug, or have a helpful suggestion to improve this documentation, [please raise it](http://jira.perfect.org:8080/servicedesk/customer/portal/1).

Before you do, please view [a comprehensive list of all open issues](http://jira.perfect.org:8080/projects/ISS/issues).

### Contributing

We welcome all contributions to and recommendations for improving the Perfect documentation. We welcome contributions to Perfect’s documentation. If you spot a typo, bug, or other errata, or have additions or suggestions to recommend, please [create a pull request or log an issue in JIRA](http://jira.perfect.org:8080/servicedesk/customer/portal/1/user/login?destination=portal%2F1).

We have [a system, written with Perfect](https://github.com/PerfectlySoft/PerfectDocGenerator), that picks up markdown files dynamically from this repository and incorporates them in an HTML docs site. This system will go live shortly.

## Table of Contents

* [Introduction](guide/introduction.md)
* [Getting Started](guide/gettingStarted.md)
* [Getting Started From Scratch](guide/gettingStartedFromScratch.md)
* [An HTTP and Web Services Primer](guide/WebServicesPrimer.md)
* [Repository Layout](guide/repositoryLayout.md)
* [Building with Swift Package Manager](guide/buildingWithSPM.md)
* [Handling Requests](guide/handlingRequests.md)
	* [Routing](guide/routing.md)
	* [HTTPRequest](guide/HTTPRequest.md)
	 	* [Using Form Data](guide/formData.md)
		* [File Uploads](guide/fileUploads.md)
	* [HTTPResponse](guide/HTTPResponse.md)
	* [Request &amp; Response Filters](guide/filters.md)
	* [JSON](guide/JSON.md)
	* [Static File Content](guide/staticFileContent.md)
	* [Mustache](guide/mustache.md)
* [Utilities](guide/utilities.md)
	* [Bytes](guide/bytes.md)
	* [File](guide/file.md)
	* [Dir](guide/dir.md)
	* [Threading](guide/thread.md)
	* [Networking](guide/net.md) @kjessup
	* [UUID](guide/UUID.md)
	* [SysProcess](guide/sysProcess.md)
	* [Log](guide/log.md)
	* [CURL](guide/cURL.md)
	* [XML](guide/xml.md)
	* [Zip](guide/zip.md)
* [Database Connectors](guide/databaseConnectors.md)
	* [SQLite](guide/SQLite.md)
	* [MySQL](guide/MySQL.md)
	* [PostgreSQL](guide/PostgreSQL.md)
	* [MongoDB](guide/MongoDB.md)
		* [MongoDB Databases](guide/MongoDB-Database.md)
		* [MongoDB Collections](guide/MongoDB-Collections.md)
		* [MongoDB Client](guide/MongoDB-Client.md)
		* [Working with BSON](guide/MongoDB-BSON.md)
	* [Redis](guide/Redis.md) @ kjessup
	* [Filemaker](guide/filemaker.md)
* [WebSockets](guide/webSockets.md) @kjessup
* [iOS Notifications](guide/iOSNotifications.md) @kjessup
* [Deployment](guide/deployment.md)
	* [Ubuntu 15.10](guide/deployment-Ubuntu1510.md)
	* Docker
	* Heroku
	* Azure
	* AWS
	* Linode
	* [Digital Ocean](guide/deployment-DigitalOcean.md)
* Platform specific Notes
	* [Ubuntu 16.04: Starting Services at System Boot](guide/starting-services.md)
