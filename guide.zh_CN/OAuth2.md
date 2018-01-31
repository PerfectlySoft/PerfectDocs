# OAuth2

Perfect Authentication OAuth2 函数库提供了 [OAuth2](https://oauth.net/2/) 编程接口以及对脸谱、谷歌和GitHub的驱动程序支持。

下列地址包括了一个详细的展示程序：
[https://github.com/PerfectExamples/Perfect-Authentication-Demo](https://github.com/PerfectExamples/Perfect-Authentication-Demo) 

## 在项目中使用

请在您的项目中修改Package.swift并增加下列依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-OAuth2.git", majorVersion: 3)
```

然后导入OAuth2模块：

``` swift
import OAuth2
```

## 配置

每个兼容OAuth2协议的厂家都会使用一个应用序号“appid”，也被称为钥匙及其密码（key / secret）。通常这个应用序号都是由厂家OAuth主机分配的，比如脸谱、谷歌和GitHub。除了应用序号之外，您还需要准备“endpointAfterAuth”（完成验证后的接入网址）和“redirectAfterAuth”（完成验证后的重定向跳转网址）用于协议接入。

比如，以下是脸谱的应用配置方法：

``` swift
FacebookConfig.appid = "yourAppID"
FacebookConfig.secret = "yourSecret"
FacebookConfig.endpointAfterAuth = "http://localhost:8181/auth/response/facebook"
FacebookConfig.redirectAfterAuth = "http://localhost:8181/"
```

以下是谷歌的应用配置方法：

``` swift
GoogleConfig.appid = "yourAppID"
GoogleConfig.secret = "yourSecret"
GoogleConfig.endpointAfterAuth = "http://localhost:8181/auth/response/google"
GoogleConfig.redirectAfterAuth = "http://localhost:8181/"
```

以下是GitHub的应用配置方法：

``` swift
GitHubConfig.appid = "yourAppID"
GitHubConfig.secret = "yourSecret"
GitHubConfig.endpointAfterAuth = "http://localhost:8181/auth/response/github"
GitHubConfig.redirectAfterAuth = "http://localhost:8181/"
```

## 追加路由

OAuth2协议依赖与一个身份验证和用户交换系统，简单说来就是用户首先从第三方获取一个身份验证的网络地址，随后在验证完成之后再返回到当前主机的新网址，以确认接受来自第三方的身份验证是否生效。

第一组路由用于跳转到OAuth2身份验证提供商，因此具体的网址都是提供商设置好的，用户通常看不到这些链接，因为整个跳转过程都是自动完成的。

第二组路由则是由身份验证提供方在完成验证手续后跳转回来的网址，也就是“endpointAfterAuth”配置选项。一旦“authResponse”（授权响应）函数运行结束则用户会自动跳转到“redirectAfterAuth”选项中配置好的新网址。

``` swift
var routes: [[String: Any]] = [[String: Any]]()

routes.append(["method":"get", "uri":"/to/facebook", "handler":Facebook.sendToProvider])
routes.append(["method":"get", "uri":"/to/github", "handler":GitHub.sendToProvider])
routes.append(["method":"get", "uri":"/to/google", "handler":Google.sendToProvider])

routes.append(["method":"get", "uri":"/auth/response/facebook", "handler":Facebook.authResponse])
routes.append(["method":"get", "uri":"/auth/response/github", "handler":GitHub.authResponse])
routes.append(["method":"get", "uri":"/auth/response/google", "handler":Google.authResponse])
```

## 第三方认证返回的信息

用户在第三方完成身份验证和授权之后，会返回给当前主机一些有关的身份信息。

注意，此时您可以采用下列方式获得当前会话的识别编码（Session ID）：

``` swift
request.session?.token
```

用户有关信息就存放在这个会话内容中：

``` swift

// 由第三方提供的用户编号
request.session?.userid

// 第三方类型——如果您的主机允许多个不同的第三方提供身份验证的话， 这个类型会很有用
request.session?.data["loginType"]

// 会话识别编码
request.session?.data["accessToken"]

// 第三方提供的用户名字
request.session?.data["firstName"]

// 第三方提供的用户姓氏
request.session?.data["lastName"]

// 第三方提供的用户头像
request.session?.data["picture"]

```

由上述信息您就可以根据需要自行将用户身份保存到自己的数据库了。

