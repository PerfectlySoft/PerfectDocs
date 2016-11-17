# log日志文件

`PerfectLogger` 模块可以用于实现将事件记录到特定文件中去。

## 使用方法

请修改您的项目文件 Package.swift 并增加以下内容：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Logger.git", majorVersion: 0, minor: 0),
```

然后您可以用 `import` 语句导入该函数库：

``` swift
import PerfectLogger
```

参考下面的示范实现同时在终端控制台和指定文件中输出事件描述：

``` swift
LogFile.debug("调试信息", "test.txt")
LogFile.info("综合消息", "test.txt")
LogFile.warning("警告信息", "test.txt")
LogFile.error("错误信息", "test.txt")
LogFile.critical("严重警告", "test.txt")
LogFile.terminal("服务器终止", "test.txt")
```

如果要将日志写入默认文件，则可以忽略第二个参数，也就是日志文件名参数。

## 定制日志文件位置

默认的日志文件名为`./log.log`。如果您希望另外选择一个位置用于存储日志文件，请在您的主程序`main.swift`中设置全局变量 `logFileLocation` ：

``` swift
logFileLocation = "/var/log/myLog.log"
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
[DEBUG] [2016-11-16 15:18:02 GMT-05:00] 测试
[INFO] [2016-11-16 15:18:02 GMT-05:00] 消息
[WARNING] [2016-11-16 15:18:02 GMT-05:00] 警告
[ERROR] [2016-11-16 15:18:02 GMT-05:00] 出错
[CRITICAL] [2016-11-16 15:18:02 GMT-05:00] 严重错误
[EMERG] [2016-11-16 15:18:02 GMT-05:00] 服务器终止
```
