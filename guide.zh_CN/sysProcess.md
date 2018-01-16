# SysProcess系统进程

在Perfect环境下，用户可以通过`SysProcess`调用系统命令，并允许在调用进程是自定义配套的参数数组和环境变量。部分进程可以立刻被执行并返回结果；其它一些进程则可以处于开通状态，随时可以进行互动性数据读写。

### 设置

请在您的"Perfect"项目中打开Package.swift设置依存关系：

``` swift
.Package(
	url: "https://github.com/PerfectlySoft/Perfect.git",
	majorVersion: 3
	)
```
在具体需要调用系统进程的源代码文件开头，声明PerfectLib库文件并增加linux下SwiftGlibc或macOS的Darwin编译条件：

``` swift
import PerfectLib

#if os(Linux)
	import SwiftGlibc
#else
	import Darwin
#endif
```

### 执行系统进程调用命令

函数方法`runProc`可以用于带参数数组进行调用系统命令，并可选择是否从命令返回中输出响应结果。

``` swift
func runProc(cmd: String, args: [String], read: Bool = false) throws -> String? {
	let envs = [("PATH", "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin")]
	let proc = try SysProcess(cmd, args: args, env: envs)
	var ret: String?
	if read {
		var ary = [UInt8]()
		while true {
			do {
				guard let s = try proc.stdout?.readSomeBytes(count: 1024) where s.count > 0 else {
					break
				}
				ary.append(contentsOf: s)
			} catch PerfectLib.PerfectError.fileError(let code, _) {
				if code != EINTR {
					break
				}
			}
		}
		ret = UTF8Encoding.encode(bytes: ary)
	}
	let res = try proc.wait(hang: true)
	if res != 0 {
		let s = try proc.stderr?.readString()
		throw  PerfectError.systemError(Int32(res), s!)
	}
	return ret
}

let output = try runProc(cmd: "ls", args: ["-la"], read: true)
print(output)
```

注意例子中的`SysProcess`命令所带环境变量并非在所有系统内通用（请用户根据宿主系统自行输入正确的环境变量以验证本例）。

### SysProcess系统进程类成员字段

#### stdin
`stdin`即当前文件系统的标准输入流。

#### stdout
`stdout` 即当前文件系统的标准输出流。

#### stderr
`stderr` 即当前文件系统的标准错误流。

#### pid
`pid` 即进程唯一标示符

### SysProcess系统进程类成员函数

#### isOpen

如果当前进程仍处于“开放”状态（即输入输出可读写），则返回真值

注意当前进程不一定处于运行状态，请用`wait(false)` 方法来检查当前进程是否处于运行状态。

``` swift
myProcess.isOpen()
```

#### close

`close` 关闭进程并清理内容。

``` swift
myProcess.close()
```

#### detatch

从进程中脱离，以确保即便是该进程进入僵尸态后资源也能够自动释放（即无需再等待进程返回）

``` swift
myProcess.detatch()
```

#### wait

判断该进程是否已经结束运行并读取其返回值。

``` swift
myProcess.wait(hang: <Bool>)
```

#### kill

终止进程并读取其返回值。

``` swift
myProcess.kill(signal: <Int32 = SIGTERM>)
```

返回值是一个`Int32`整型代码
