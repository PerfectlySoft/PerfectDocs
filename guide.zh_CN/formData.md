# 使用表单

在一个REST应用程序中，经常会使用到几个HTTP“请求动作”。通常是“GET”、“POST”

>上述请求的详细用法已经超过本文范围。

一个HTTP的“GET”请求是通过URL连接中直接以参数方式传递的：

```
http://www.example.com/page.html?message=Hello,%20World!
```

以上的“GET参数”可以通过`.queryParams`方法来获取

``` swift
let params = request.queryParams
```

以上的例子只适合GET请求，而`.queryParams`方法能够应用到任何一种包含查询参数的HTTP请求。

## POST参数

POST参数是用于在浏览器和服务器之间传递复杂数据的标准方法。

Perfect的HTTP函数库可以为用户以数组形式访问POST参数提供便利。

为了从查询和POST请求中获取所有`[(String,String)]`参数数组，请使用下面的方法：

``` swift
let params = request.params()
```

如果只需要返回POST的`[(String,String)]`参数数组：

``` swift
let params = request.postParams()
```

如果需要根据一个具体名称（比如多个checkbox选项表）返回所有的参数，请使用：

``` swift
let params = request.postParams(name: <String>)
```

这会返回一个字符串数组：`[String]`

如果需要返回一个特定参数，可以输入选择一个可选的`String?`字符串值

``` swift
let param = request.param(name: <String>)
```

在`request` 对象中整理POST参数时，如果需要填写一个具体参数但是客户端表单并没有按要求填写，此时为该参数设定一个可选的`String?`字符串默认值会非常有用：

``` swift
let param = request.param(name: <String>, defaultValue: <string>)
```
