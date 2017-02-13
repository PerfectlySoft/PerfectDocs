## CSRF (跨网站请求伪造) 安全功能

跨网站请求伪造是一种网络攻击方法，能够在用户不知情的情况下代理用户已经完成的身份验证在目标网站上执行非法操作。 **CSRF 攻击主要瞄准状态变更操作，而不是盗取数据，因为攻击者无法看到来源请求的响应结果。** 此类攻击一般利用社交媒体设下陷阱（比如通过邮件或者聊天发送链接），通过引诱用户点击链接执行攻击者的操作。典型的受害者可能在不知情的情况下被转移资金，或者改变了邮件地址等等。如果受害者是一个具有高度权限的账户，比如管理员账户，则整个网站都可能被劫持。[1] - (OWASP)

CSRF 是一种经常被忽视的攻击方法，因此常常造成混乱，除非在整个软件体系上加以防范。一旦在最高级别的体系上实施防范，则网站整体安全性会得到大大提高。

[Perfect Sessions](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/sessions.md) 会话模块包括了对 CSRF 的配置方法。

如果您的工程软件配置文件 Package.Swift 包括了 Perfect Sessions 会话管理，或者任何带有数据源驱动的应用实现，则您的工程已经包括CSRF支持。

## 相关范例

* [Perfect-Session-Memory-Demo](https://github.com/PerfectExamples/Perfect-Session-Memory-Demo)


## 配置

一个典型的 CSRF 配置可能看起来像这样

``` swift 
SessionConfig.CSRF.checkState = true
SessionConfig.CSRF.failAction = .fail
SessionConfig.CSRF.checkHeaders = true
SessionConfig.CSRF.acceptableHostnames.append("http://www.example.com")
SessionConfig.CSRF.requireToken = true
```

### SessionConfig.CSRF.checkState

这是总开关，如果开启，则CSRF将在所有路由上启动安全管制。

### SessionConfig.CSRF.failAction

该选项用于处理 CSRF 认证失败时应该采取的操作，允许值为：

* `.fail` - 执行暂停。服务器将停止处理任何新请求操作，而HTTP状态会变更为 `406 Not Acceptable`。
* `.log` - 允许继续服务，但是事件将记录到日志中去。
* `.none` - 允许继续服务，而且也不会采取任何附加操作。

### SessionConfig.CSRF.acceptableHostnames

该数组用于代表允许进行CSRF的主机名称。


### SessionConfig.CSRF.checkHeaders

如果 `CORS.checkheader` 被启动（配置为 `true`）请求来源和主机名将被验证。

* 请求头数据必须包括 `Origin` 和 `Referrer` 或 `X-Forwarded-For` 三个内容段。
* 如果 "origin" 值已经在配置选项 `SessionConfig.CSRF.acceptableHostnames`名单中，则 CSRF 检查则认可该来源，结束认证并执行请求后续操作。
* 请求必须包括 `Host` 或 `X-Forwarded-Host`，用于代表代理主机 ("host").
* "host" 和 "origin" 值必须完全匹配。



### SessionConfig.CSRF.requireToken

当设置为真时，该设置会要求所有 HTTP POST 包括一个 "_csrf" 参数，或者如果内容类型为"application/json"时请求头数据必须包括一个相关的 "X-CSRF-Token" 票据变量。请求的内容或参数必须和`request.session.data["csrf"]`数值匹配。该数值在会话开始时自动设置。

### 会话状态

虽然不是一个配置选项，但是需要注意的时，如果总开关`SessionConfig.CSRF.checkState`设置为真，则如果会话是一个“新会话”，则不会允许任何POST请求。这是根据安全建议中特别设置的特征。


 


[1] - OWASP, Cross-Site Request Forgery (CSRF): [https://www.owasp.org/index.php/Cross-Site\_Request\_Forgery\_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))