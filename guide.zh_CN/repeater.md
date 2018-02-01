# Perfect Repeater

本函数库提供了实现定期运行程序的方法。

## 示范代码

* [Perfect-Repeater-Example](https://github.com/PerfectExamples/Perfect-Repeater-Demo)

## 快速上手

使用本函数库需要调用PerfectLib模块。此外，请追加Perfect-Repeater模块到您项目的Package.swift文件中去：

``` swift
.Package(url:"https://github.com/PerfectlySoft/Perfect-Repeater.git", majorVersion: 3)
```

## 使用方法

首先在源代码中导入函数库：

``` swift
import PerfectRepeater
```

然后您就可以在程序中使用如下定时器：

``` swift
Repeater.exec(timer: <Double>, callback: <Closure>)
```

其中，`timer`间隔时间，单位是秒。

`callback` 为期望定期执行的回调函数句柄，返回值必须是布尔类型。您自行定义的这个回调函数句柄的返回值在于，如果返回为真则表示定时器仍然有效，将继续按时调用；否则如果返回假，表示自动停止继续执行，定时器将把该句柄从执行队列中移除。

下列代码展示了定时器的使用，并演示了在按期执行若干次后，如何自动停止：

``` swift
var opt = 1

let c = {
	() -> Bool in
	print("XXXXXX")
	return true
}
let cc = {
	() -> Bool in
	print("你好！ (\(opt))")
	if opt < 10 {
		opt += 1
		return true
	} else {
		print("定时器结束")
		return false
	}
}

Repeater.exec(timer: 3.0, callback: c)
Repeater.exec(timer: 2.0, callback: cc)
```