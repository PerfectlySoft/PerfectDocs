# Perfect-SPNEGO [English](README.md)

该项目实现了一个用于 Swift 服务器编程的 SPNEGO 安全机制函数库

### 开始之前

Perfect SPNEGO 瞄准的是所有用Swift 编程的通用服务器组件，因此能够应用于⚠️任何⚠️服务器，比如 HTTP / FTP / SSH，等等。

尽管该服务器组件能够原生支持 Perfect 自己的 HTTP 服务器，但是实际上也适用于任何一种其他互联网协议的服务器。

在安装到目标服务器软件之前，请确认您的服务器已经完成了Kerberos V的配置。

### Xcode 编译指南

如果您希望使用Xcode 编译本项目，请使用将特殊的符号传递给 SPM 软件包管理器：

```
$ swift package -Xlinker -framework -Xlinker GSS generate-xcodeproj
```

### Linux 编译指南

函数库 libkrb5-dev 需要在编译之前安装：

```
$ sudo apt-get install libkrb5-dev
```

如果您的服务器是一个KDC（安全控制中心），您可以忽略这一步，否则请安装 Kerberos V5 工具组件，以及开发函数库：

```
$ sudo apt-get install krb5-user
```

### KDC配置

请配置好您的应用服务器/etc/krb5.conf，以便于该服务器能够正常连接到目标的KDC。请参考下面的例子，在这个例子中，应用服务器希望连接到控制区域`KRB5.CA`，并且该控制区域的KDC控制中心域名为`nut.krb5.ca`:

```
[realms]
KRB5.CA = {
	kdc = nut.krb5.ca
	admin_server = nut.krb5.ca
}
[domain_realm]
.krb5.ca = KRB5.CA
krb5.ca = KRB5.CA
```

### 准备Kerberos 授权钥匙

下一步需要联系您的KDC 管理员，获得应用服务器将使用的keytab钥匙文件。

参考下面的例子的示范配置，⚠️下列所有主机必须注册到同一个DNS⚠️

- KDC 安全控制中心服务器: nut.krb5.ca
- 应用服务器: coco.krb5.ca
- 应用服务器计划安装的协议类型: HTTP

如果处于上述环境，则KDC管理员需要登录`nut.krb5.ca` 控制区域并采取下列操作：

```
kadmin.local: addprinc -randkey HTTP/coco.krb5.ca@KRB5.CA
kadmin.local: ktadd -k /tmp/krb5.keytab HTTP/coco.krb5.ca@KRB5.CA
```

生成钥匙文件krb5.keytab后，请将该钥匙安全地传输到您的应用服务器`coco.krb5.ca`并将文件移动到目录`/etc`下，然后赋予其适当权限，以便于您的服务器可以访问到。

## 快速开始

首先在您的软件工程配置文件Package.swift中增加下列依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-SPNEGO.git", majorVersion: 1)
```

然后将 Perfect-SPNEGO 引入源代码：

``` swift
import PerfectSPNEGO
```

### 在应用程序中连接到 KDC

请使用默认钥匙文件 `/etc/krb5.keytab` 中列出的服务器名称，然后在程序中注册到KDC：

``` swift
let spnego = try Spnego("HTTP@coco.krb5.ca")
```

请注意主机名和协议⚠️务必⚠️与钥匙文件中的内容保持完全一致。

### 响应 SPNEGO 挑战

初始化成功之后，您的应用程序即可使用`spnego`对象响应此类挑战。比如，如果用户正尝试访问应用服务器：

```
$ kinit rocky@KRB5.CA
$ curl --negotiate -u : http://coco.krb5.ca
```

这种情况下，curl命令行将在其HTTP请求头数据中发送一个 base64 编码的申请表：

```
> Authorization: Negotiate YIICnQYGKwYBBQUCoIICkTCCAo2gJzAlBgkqhkiG9xIBAgI ...
```

您的服务器一旦收到这个申请表，即可使用 `spnego` 对象进行解码分析：
```
let (user, response) = try spnego.accept(base64Token: "YIICnQYGKwYBBQUCoIICkTCCAo2gJzAlBgkqhkiG9xIBAgI...")
```

如果成功的话，用户名称将会是"rocky@KRB5.CA"。而变量`response`有可能为空，表示这种申请不需要回复多余的数据；否则请将这个同样是base64的字符串返回给客户。

到此为止，您的程序已经获得了用户信息，您的应用程序可以根据这个验证过的用户名称，查看ACL（安全控制表）来决定是否允许该用户访问其期待的资源。

## 有关示例

以下连接包括了一个如何在 Perfect HTTP 服务器中应用 SPNEGO 安全机制的范例：
* [Perfect-Spnego-Demo](https://github.com/PerfectExamples/Perfect-Spnego-Demo)