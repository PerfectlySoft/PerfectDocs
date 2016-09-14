# CURL联网传输

Perfect为开源cURL库提供了一组函数，用于调用cURL命令行工具进行标准协议支持的网络文件传输

cURL采用URL语法传输数据，并支持下列协议：

- DICT
- FILE
- FTP
- FTPS
- Gopher
- HTTP
- HTTPS
- IMAP
- IMAPS
- LDAP
- LDAPS
- POP3
- POP3S
- RTMP
- RTSP
- SCP
- SFTP
- SMTP
- SMTPS
- Telnet
- TFTP

cURL内建支持SSL安全认证，HTTP POST，HTTP PUT，FTP上载，网页代理proxies，cookies，用户名＋密码认证，proxy tunnelling代理隧道，等等。

### 使用前准备

除PerfectLib 之外，还需要在Package.swift文件中说明当前项目对Perfect-cURL的依存关系：

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect-Curl.git",
	majorVersion: 2, minor: 0
	)
```

在代码中如果需要调用cURL类函数，请在代码中首先声明库函数信息：

``` swift
import PerfectCURL
```

### Linux版本注意事项

请确保目标操作系统已经安装了libcurl。

```
sudo apt-get install libcurl-dev
```

## 使用Perfect cURL

初始化一个cURL操作：

``` swift
let curlObject = CURL(url: "http://www.perfect.org")
```

当前cURL对象的URL路径属性可以通过`.url`方法进行读写设置：

``` swift
print(curlObject.url)
>> http://www.perfect.org
```

### 设置cURL选项

使用cURL前必须为其选择适当的属性。多数选项已经超出了本文范围。如果您需要详细了解一个完整的cURL选项说明，请参考Perfect API编程参考手册，或参考[libcurl API 参考（英文版）](https://curl.haxx.se/libcurl/c/)

为设置具体的选项，请为`.setOption`挑选适当的变量，如下所示：

如果选项值为整型或64位整型（Int或Int64）：

``` swift
curlObject.setOption(<OPTION>, int: <INT>)
```

如果选项值为指针类型：

``` swift
curlObject.setOption(<OPTION>, v: <UnsafeMutablePointer<Void>>)
```
> 请注意指针所指向的值不会被复制，也不可进行其它操作或保存。请程序员自行确保其指针变量的声明周期（初始化、内存分配和释放）以及相应用法

如果选项为回调函数：

``` swift
curlObject.setOption(<OPTION>, f: <curl_func>)
```

如果选项值为字符串：

``` swift
curlObject.setOption(<OPTION>, s: <String>)
```

比如，当需要关闭SSL点对点认证时，请使用：

``` swift
curlObject.setOption(CURLOPT_SSL_VERIFYPEER, int: 0)
```

### 执行cURL操作并返回数据

当所有选项设置完毕时，就可以采取如下方法之一进行文件传输操作了：

执行当前传输请求（）：

``` swift
var perf = curlObject.perform()
```

执行后返回一个序列，内容如下：

* `Bool` - 是否需要再次调用perform()
* `Int` - 返回值代码
* `[UInt8]` - 如果存在目标请求的头数据的话，以字节数组形式返回头
* `[UInt8]` - 如果存在目标请求的内容数据字节的话，以字节数组形式返回内容数据

如果需要以非阻塞方式调用cURL的perform方法，可以采用下面的方式调用闭包，其中返回值，头数据和内容数据作为参数执行：

``` swift
var perf = curlObject.perform { <Int>, <[UInt8]>, <[UInt8]> }
```

下面举例调用perform，采用闭包方式读取数据

```swift
curlObject.perform {
    code, header, body in

    print("请求错误代码：\(code)")
    print("服务器响应代码：\(curlObject.responseCode)")
    print("返回头数据：\(header)")
    print("返回内容数据：\(body)")
}

```

如有必要执行一个阻塞当前进程直至所有查询结果返回的完整cURL请求，可以使用`.performFully` 方法：

```swift
var perf = curlObject.performFully()
```

调用后返回序列为

* `Int` - 返回值代码
* `[UInt8]` - 如果存在目标请求的头数据的话，以字节数组形式返回头
* `[UInt8]` - 如果存在目标请求的内容数据字节的话，以字节数组形式返回内容数据


### 如果请求执行后返回一个序列内容，读取cURL响应数据的方法

cURL响应内容可以按如下返回的样本序列进行读取：

``` swift
let curlObject = CURL(url: "http://www.perfect.org")

curlObject.setOption(CURLOPT_SSL_VERIFYPEER, int: 0)

var header = [UInt8]()
var body = [UInt8]()

var perf = curlObject.perform()
while perf.0 {
	if let h = perf.2 {
		header.append(contentsOf: h)
	}
	if let b = perf.3 {
		body.append(contentsOf: b)
	}
	perf = curlObject.perform()
}
if let h = perf.2 {
	header.append(contentsOf: h)
}
if let b = perf.3 {
	body.append(contentsOf: b)
}
let perf1 = perf.1

let response = curlObject.responseCode
print(response == 200, "\(response)")

print("Header size: \(header.count)")
print("Body size: \(body.count)")
```

### cURL信息和错误代码

给定`CURLINFO`字符串值后调用`.getInfo`方法可获得相应信息：

``` swift
let (s:String, c:CURLCode) = curlObject.getInfo(<CURLINFO>)
let (i:Int, c:CURLCode) = curlObject.getInfo(<CURLINFO>)
```

关于`CURLINFO`的详细清单和完整描述请参阅：[cURL信息选项（英文）](https://curl.haxx.se/libcurl/c/easy_getinfo_options.html)

如果需要从cURL返回代码中获取相应的字符串信息，请调用`.strError`方法：

```swift
let errMsg = curlObject.strError(code: <CURLcode>)
```

### 重置和关闭cURL对象

如果需要清理并关闭cURL对象，并用于再次调用，请使用`.reset()`方法

```swift
curlObject.reset()
```

如果cURL对象创建后不需要再使用，请调用`.close()`方法进行彻底关闭

```swift
curlObject.close()
```
