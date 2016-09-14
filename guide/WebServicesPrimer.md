# HTTP和Web服务基础

大多数已知的Web开发都是在服务器端基于几种主流技术实现的。

本文将介绍这些内容，因为这些内容与我们的Perfect（服务器端Swift语言）项目息息相关。此外，本文还将介绍一些应用编程接口（API）开发的基本概念。

### 什么是API?

API是连接两个软件系统之间的桥梁。通常API会接受标准类型作为输入，并在内部转换然后与其它系统工作实现具体的软件操作，或者返回具体的信息。

一个API的例子是GitHub网站的API。大多数人可能很熟悉GitHub网站的页面，但是其实GitHub还有一个API。该API允许像[Jira](https://www.atlassian.com/software/jira")这样的应用程序执行任务并发布管理系统，或者像[Bamboo](https://www.atlassian.com/software/bamboo)这样的持续集成系统（CI），能够直接连接到您的代码资源库内以 **为任何单独的系统自身提供更为强大的功能。**

这样的API通常都建立在HTTP或HTTPS基础上

### HTTP and HTTPS

“HTTP”是“超文本传输协议”的缩写。这是一组用于在互联网上传输、交换信息和发布多媒体网页的标准。

“HTTPS”是HTTP的安全加密通信协议。通信的双方通过一个双方都信任的“钥匙”来为通信内容加密解密。安全机制越复杂，其内容被第三方截获、破译和篡改的可能性越低。

通常用浏览器访问网站的时候，大部分时间都是通过HTTP完成的；如果您看到浏览器有一个锁的标志，那么应该用的是HTTPS协议。

当一个iOS或Android应用程序去访问后台服务器信息时——比如获取天气预报——可能用的是HTTP，但更常见的情况是使用HTTPS。

## API的通用形式

### 路由

一个API可以包含多个“路由”，每个“路由”看起来就像文件系统中的一个目录或文件一样。每个路由节点都可以执行不同的操作。比如一个“查看用户清单”的路由与一个“创建新用户”的路由肯定是不一样的。

在一个API中，这些路由往往不是在真实的目录和文件下存在的，而是指向了一些函数。这些“目录结构”暗示了这些函数的逻辑分组。比如：

```
/api/v1/users/list
/api/v1/users/detail
/api/v1/users/create
/api/v1/users/modify
/api/v1/users/delete

/api/v1/companies/list
/api/v1/companies/detail
/api/v1/companies/create
/api/v1/companies/modify
/api/v1/companies/delete
```

这个例子描述了一个典型的“CRUD”系统——即Create创建、Read读取、Update更新和Delete删除。两组内容函数的区别貌似仅仅是在`users`用户和`companies`公司着两个实体上，但是其实它们指向的是完全不同的函数。

此外，这些“detail”、“modify”和“delete”路由往往会带有一个记录的唯一标示符。比如：

```
/api/v1/users/list
/api/v1/users/detail?id=5d096846-a000-43db-b6c5-a5883135d71d
/api/v1/users/create
/api/v1/users/modify?id=5d096846-a000-43db-b6c5-a5883135d71d
/api/v1/users/delete?id=5d096846-a000-43db-b6c5-a5883135d71d
```

上述例子说明了传递给服务器的这些路由；id参数关联了一个特定的记录。创建函数create和列表函数list不需要id因为id对于这些路由来说是无关的。

除了这些路由之外，发向这些路由的每一个请求都会包括一个HTTP的“动作”。

### HTTP动作

HTTP动作是一个网页浏览器或移动应用端向路由发出请求的额外信息。这些动作能够向API服务器提供数据接收时需要的“上下文”信息。

常见HTTP动作包括

#### GET
GET方法向特定路由发出请求，其中从客户端发向服务器的所有参数包含在URL字符串中。使用GET方法发出的请求只能接收数据，不能由其它作用。使用GET请求来删除数据库的一条记录是可行的，但是不推荐这么做。

#### POST
POST方法通常用于向数据库内发送创建一条新记录的表单，比如在一个网站上填写好一个表单后提交给服务器。每个表单中都有成对出现的字段名和字段值，便于API服务器读取这些变量信息。

#### PATCH
PATCH方法整体上来说和POST差不多，但是用于更新记录，而不是新建记录。

#### PUT
PUT方法主要用于上传文件，一个典型的PUT请求通常会在其消息体内部包含一个即将发给服务器的文件。

#### DELETE
DELETE请求用于通知API服务器删除特定资源。通常在其URL中会包含一个唯一的标识信息。



### 使用HTTP动作和路由来简化一个API

当综合使用时，每一个HTTP请求的这两个组成部分能够有效降低API的复杂性。

查看以下包含了HTTP动作的URL结构，您会发现其内容和形式都要简单多了、也更明确了：

```
GET     /api/v1/users
GET     /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
POST    /api/v1/users
PATCH   /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
DELETE  /api/v1/users/5d096846-a000-43db-b6c5-a5883135d71d
```

上述例子给人第一感觉就是都指向了同一个路由：`/api/v1/users`。但是每一个路由都执行一个不同的操作。类似的，头两个GET路由的区别是第二个GET的后缀带了一个用户id编码作为参数，与之前的例子有所差别。这种命令通常会被一个在路由设置中的“通配符”获取。

简单了解上述概念之后，请查看Perfect文档中的《Handling Requests处理请求》有关章节。这些内容有助于理解如何应用路由指向具体的函数和方法，如何将访问从客户端（无论是浏览器还是移动应用）传递给服务器的变量信息。
