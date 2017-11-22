# log日志文件

`PerfectLogger` 模块可以用于实现将事件记录到特定文件中去。

## 使用方法

请修改您的项目文件 Package.swift 并增加以下内容：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Logger.git", majorVersion: 3),
```

然后您可以用 `import` 语句导入该函数库：

``` swift
import PerfectLogger
```

参考下面的示范实现同时在终端控制台和指定文件中输出事件描述：

``` swift
LogFile.debug("调试信息", logFile: "test.txt")
LogFile.info("综合消息", logFile: "test.txt")
LogFile.warning("警告信息", logFile: "test.txt")
LogFile.error("错误信息", logFile: "test.txt")
LogFile.critical("严重警告", logFile: "test.txt")
LogFile.terminal("服务器终止", logFile: "test.txt")
```

如果要将日志写入默认文件，则可以忽略第二个参数，也就是日志文件名参数。

## 在事件之间通过设置"eventid"进行关联

每个日志记录的事件都能返回一个事件的id标识符。将事件id标识符应用到新的事件上最直接的好处就是可以实现各个事件之间的关联关系，比如：

``` swift
let eid = LogFile.warning("test 1")
LogFile.critical("test 2", eventid: eid)
```

则在日志中的记录会变成：

```
[WARNING] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] test 1
[CRITICAL] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] test 2
```

返回的结果被标记为可丢弃结果 `@discardableResult` （swift 编译标志），因此如果没有重复使用的需要时，可以安全地忽略。


## 定制日志文件位置

默认的日志文件名为`./log.log`。如果您希望另外选择一个位置用于存储日志文件，请设置变量 `LogFile.location`：

``` swift
LogFile.location = "/var/log/myLog.log"
```
这样上述消息就可以直接写入文件了：

``` swift
LogFile.debug("调试")
LogFile.info("消息")
LogFile.warning("警告")
LogFile.error("出错")
LogFile.critical("严重错误")
LogFile.terminal("服务器终止")
```

## 输出效果

```
[DEBUG] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] 调试
[INFO] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] 消息
[WARNING] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] 警告
[ERROR] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] 出错
[CRITICAL] [62f940aa-f204-43ed-9934-166896eda21c] [2016-11-16 15:18:02 GMT-05:00] 严重错误
[EMERG] [ec6a9ca5-00b1-4656-9e4c-ddecae8dde02] [2016-11-16 15:18:02 GMT-05:00] 服务器终止
```
