# 日志

Perfect包含了一个内建的错误日志系统。用户可以调用log进行日志分级记录。每个级别的日志都可以输出到命令行或者系统日志。

内建的日志警告级别包括（依严重级别依次递增排序）：

* **debug:** 调试 `[DBG]`
* **info:** 信息 `[INFO]`
* **warning:** 警告 `[WARN]`
* **error:** 错误 `[ERR]`
* **critical:** 严重错误 `[CRIT]`
* **terminal:** 服务终止 `[TERM]`

### 如果需要将日志信息直接输出到命令行

```swift
Log.debug(message: "程序第123行: value \(myVar)")
Log.info(message: "程序第123行:")
Log.warning(message: "调用错误句柄")
Log.error(message: "满足错误条件\(errorMessage)")
Log.critical(message: "发现异常：\(exceptionVar)")
Log.terminal(message: "异常失控，服务终止。\(infoVar)")
```

### 如果需要将日志信息输出到系统日志

如果需要将所有日志结果输出到系统日志中去，请在程序启动的配置过程中调用`SysLogger()`并设置`Log.logger`属性。一旦设置完成，所有日志将输出到系统日志文件，同时在命令行中同步显示。

```swift
Log.logger = SysLogger()
```

如果您希望将日志渠道停止输出到系统并返回到命令行，请随时调用上述方法将属性设置回`ConsoleLogger()`

```swift
Log.logger = ConsoleLogger()
```
