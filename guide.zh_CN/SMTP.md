# Perfect – SMTP 简单邮件协议

本项目提供了SMTP函数库封装。

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分。

请确保您已经安装并激活了最新版本的 Swift 4.0 工具链。

## Linux 编译注意事项

请确保编译之前您的 Ubuntu 16.04 上已经安装了 `libssl-dev` 程序库：

```
sudo apt-get install libssl-dev
```
## 快速上手

使用 SMTP 库函数之前，请首先修改您的项目 Package.swift 文件并增加如下依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-SMTP.git", majorVersion: 3)
```

随后在源代码开始部分增加导入说明：

``` swift
import PerfectSMTP
```

## 主要数据结构

Perfect SMTP 包含三类数据结构： SMTPClient, Recipient and EMail.

### SMTPClient

SMTPClient 对象用于存储连接到邮件服务器的登录信息，比如下列例子：

``` swift
let client = SMTPClient(url: "smtp://邮件服务器域名地址", username: "某人@某地址", password:"密码")
```

### Recipient

Recipient 是用于处理用户名和邮件地址标准化的一个数据结构：

``` swift
let recipient = Recipient(name: "收件人/寄件人全名", address: "某人@某网络")
```

### EMail

EMail 对象用于起草和发送邮件，参考如下例子：

``` swift
// 起草一封邮件，首先用登录信息初始化邮件
var email = EMail(client: client)

// 设置标题
email.subject = "邮件标题"

// 设置寄件人信息
email.from = Recipient(name: "寄件人姓名", address: "我@电邮地址")

// 邮件正文，可以是HTML格式，也可以是普通文本
email.html = "<h1>Hello, world!</h1><hr><img src='http://www.perfect.org/images/perfect-logo-2-0.svg'>"

// 设置收件人，包括收件人、抄送和秘密抄送
email.to.append(Recipient(name: "收件人全名", address: "某人@某地址"))
email.cc.append(Recipient(name: "抄送", address: "某人@某其他地址"))
email.bcc.append(Recipient(name: "密送", address: "某人@某处"))

// 追加附件
email.attachments.append("/本地/计算机/文件.txt")
email.attachments.append("/本地/电脑/照片.jpg")

// 发送邮件，发送结束后回调
do {
  try email.send { code, header, body in
    /// 打印邮件服务器响应信息
    print(code)
    print(header)
    print(body)
  }//end send
}catch(let err) {
  /// 如果程序运行到了这里，表示发送不成功
}
```

#### EMail 对象的成员变量和成员函数

- client: SMTPClient，邮件服务器登录信息
- to: [Recipient]，收件人数组
- cc: [Recipient]，抄送人地址数组
- bcc:[Recipient]，密送人地址数组
- from: Recipient，当前寄件人地址
- subject: String，邮件标题
- attachments: [String]，邮件附件文件名构成的数组，比如 ["/本地/计算机/文件.txt", "/本地/电脑/照片.jpg" ...]
- content: String，邮件正文，可以是普通文本，或者是HTML
- html: String，邮件正文的别名，和content 使用相同的变量
- text: String，将内容设置为纯文本
- send(completion: @escaping ((Int, String, String)->Void))，邮件发送函数，参数为回调函数。
回调函数包括以下三个参数，详细含义请参考 Perfect-CURL `performFully()`函数：
  - code: Int，邮件服务器响应代码，正常值是零。
  - header: String，邮件响应头数据字符串。
  - body: String，邮件响应体数据字符串。


## 总结

以下是发送邮件过程的简明结构：

``` swift
import PerfectSMTP

let client = SMTPClient(url: "smtp://smtp.gmx.com", username: "yourname@youraddress.com", password:"yourpassword")

var email = EMail(client: client)

email.subject = "邮件主题"
email.content = "邮件内容"

email.cc.append(Recipient(address: "who@where.com"))

do {
  try email.send { code, header, body in
    /// 从邮件服务器响应的结果
    print(code)
  }//end send
}catch(let err) {
  /// 出错了
}
```

## 示范程序

在github上有一个完整邮件发送程序供参考：
[Perfect SMTP Demo](https://github.com/PerfectExamples/Perfect-SMTP-Demo)

## SMTPS协议注意事项

我们收到了很多关于google smtp示范的要求，在此感谢 @ucotta 、@james，当然还有来自Perfect 官方专业支持的 @iamjono，以下说明可能会为您的gmail客户端开发有所帮助： ⚠️*SMTPClient 的地址应该设置为 `smtps://smtp.gmail.com:465`，并且在谷歌设置中一定要开启“降低安全性以便于某些传统客户端访问”*⚠️

以下是使用SMTPS加密协议的示范例子（以gmail为例）

``` swift
import PerfectSMTP

let client = SMTPClient(url: "smtps://smtp.gmail.com:465", username: "yourname@gmail.com", password:"yourpassword")

var email = EMail(client: client)

email.subject = "a topic"
email.content = "a message"

email.cc.append(Recipient(address: "who@where.com"))

do {
  try email.send { code, header, body in
    /// response info from mail server
    print(code)
  }//end send
}catch(let err) {
  /// something wrong
}
```
