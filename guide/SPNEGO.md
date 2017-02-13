# Perfect-SPNEGO [简体中文](README.zh_CN.md)

This project provides a general server library which provides SPNEGO mechanism.

### Before Start

Perfect SPNEGO is aiming on a general application of Server Side Swift, so it could be plugged into *ANY* servers, such as HTTP / FTP / SSH, etc.

Although it supports Perfect HTTP server natively, it could be applied to any other Swift servers as well.

Before attaching to any actual server applications, please make sure your server has been already configured with Kerberos V5.

### Xcode Build Note

If you would like to use Xcode to build this project, please make sure to pass proper linker flags to the Swift Package Manager:

```
$ swift package -Xlinker -framework -Xlinker GSS generate-xcodeproj
```

### Linux Build Note

A special library called libkrb5-dev is required to build this project:

```
$ sudo apt-get install libkrb5-dev
```

If your server is a KDC, then you can skip this step, otherwise please install Kerberos V5 utilities:

```
$ sudo apt-get install krb5-user
```


### KDC Configuration

Configure the application server's /etc/krb5.conf to your KDC. The following sample configuration shows how to connect your application server to realm `KRB5.CA` under control of a KDC named `nut.krb5.ca`:

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

### Prepare Kerberos Keys for Server
Contact to your KDC administrator to assign a `keytab` file to your application server.

Take example, *SUPPOSE ALL HOSTS BELOW REGISTERED ON THE SAME DNS SERVER*:

- KDC server: nut.krb5.ca
- Application server: coco.krb5.ca
- Application server type: HTTP

In such a case, KDC administrator shall login on `nut.krb5.ca` then perform following operation:

```
kadmin.local: addprinc -randkey HTTP/coco.krb5.ca@KRB5.CA
kadmin.local: ktadd -k /tmp/krb5.keytab HTTP/coco.krb5.ca@KRB5.CA
```

Then please ship this krb5.keytab file securely and install on your application server `coco.krb5.ca` and move to folder `/etc`, then grant sufficient permissions to your swift application to access it.

## Quick Start

Add the following dependency to your project's Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-SPNEGO.git", majorVersion: 1)
```

Then import Perfect-SPNEGO to your source code:

``` swift
import PerfectSPNEGO
```

### Connect to KDC

Use the key in your default keytab `/etc/krb5.keytab` file on application server to register your application server to the KDC.

``` swift
let spnego = try Spnego("HTTP@coco.krb5.ca")
```

Please note that the host name and protocol type *MUST* match the records listed in the keytab file.

### Respond to A Spnego Challenge

Once initialized, object `spnego` could respond to the challenges. Take example, if a user is trying to connect to the application server as:

```
$ kinit rocky@KRB5.CA
$ curl --negotiate -u : http://coco.krb5.ca
```

In this case, the curl command would possibly send a base64 encoded challenge in the HTTP header:

```
> Authorization: Negotiate YIICnQYGKwYBBQUCoIICkTCCAo2gJzAlBgkqhkiG9xIBAgI ...
```

Once received such a challenge, you could apply this base64 string to the `spnego` object:
```
let (user, response) = try spnego.accept(base64Token: "YIICnQYGKwYBBQUCoIICkTCCAo2gJzAlBgkqhkiG9xIBAgI...")
```

If succeeded, the user would be "rocky@KRB5.CA". The variable `response` might be `nil` which indicates nothing is required to reply such a token, otherwise you should send this `response` back to the client.

Up till now, your application had already got the user information and request, then the application server might decide if this user could access the objective resource or not, according to your ACL (access control list) configuration.

## Relevant Examples

A good example to demonstrate how to use SPNEGO in a Perfect HTTP Server can be found from:
* [Perfect-Spnego-Demo](https://github.com/PerfectExamples/Perfect-Spnego-Demo)