# Perfect SysInfo


本项目提供了一个用于系统性能指标监控的函数库。

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分，但也可以独立使用。

请确保您的系统已经安装了Swift 4.0工具链。


## 快速上手

首先请在您的 Package.swift 文件中增加依存关系：

``` swift

.Package(url: "https://github.com/PerfectlySoft/Perfect-SysInfo.git", majorVersion: 3)

```

在源程序中导入函数库：

``` swift

import PerfectSysInfo

```

之后您就可以使用SysInfo 类函数了。

### CPU 用量

调用系统静态变量 `SysInfo.CPU` 可以获得所有处理器内核用量，返回结果为字典形式`[String: [String: Int]]`，举例如下：

``` swift

print(SysInfo.CPU)

// 典型的单核处理器统计结果（Linux）：

[
  "cpu0":
    ["nice": 1201, "system": 3598, "user": 8432, "idle": 8657606],
  "cpu":
    ["nice": 1201, "system": 3598, "user": 8432, "idle": 8657606]
]

// 下面是另外一个例子，8核处理器统计结果（Mac OS）：
[
  "cpu3":
    ["user": 18095, "idle": 9708265, "nice": 0, "system": 16177],
  "cpu5":
    ["user": 18032, "idle": 9708329, "nice": 0, "system": 16079],
  "cpu7":
    ["user": 18186, "idle": 9707892, "nice": 0, "system": 16285],
  "cpu":
    ["user": 344301, "idle": 9201762, "nice": 0, "system": 196763],
  "cpu0":
    ["user": 730263, "idle": 8387000, "nice": 0, "system": 626684],
  "cpu2":
    ["user": 648287, "idle": 8799969, "nice": 0, "system": 294749],
  "cpu1":
    ["user": 17708, "idle": 9708996, "nice": 0, "system": 15950],
  "cpu4":
    ["user": 647701, "idle": 8800643, "nice": 0, "system": 294544],
  "cpu6": ["user": 656136, "idle": 8793002, "nice": 0, "system": 293640]
]

```

返回的字典为 `N+1` 组数据，其中 N是内核数，命名分别为"cpu0"、"cpu1" ... "cpu N-1"，单独的命名"cpu"为所有内核平均综合结果。
每组数据都会包括 `idle`（空闲）、`user`（用户）、`system`（系统）和`nice`（调度）四个统计值。通常`idle`应该尽量大一些，表示系统不是那么繁忙。

### 内存用量

调用静态属性 `SysInfo.Memory` 会返回另外一个字典`[String: Int]` 用于表示内存用量，单位是 ** MB （兆字节）**:

``` swift

print(SysInfo.Memory)

```

** 注意 ** 由于该信息会因操作系统而表述方法不一样，因此请在调用前用条件编译选项`#if os(Linux) #else #endif`来判定操作系统类型。每个指标定义已经超出本文的范围，详细内容请参考具体的操作系统使用手册。

比如调用上述命令后，一个典型的Linux系统将输出如下报告（1G内存，大约599兆可用）：

``` swift
[
  "Inactive": 283, "MemTotal": 992, "CmaFree": 0,
  "VmallocTotal": 33554431, "CmaTotal": 0, "Mapped": 74,
  "SUnreclaim": 14, "Writeback": 0, "Active(anon)": 98,
  "Shmem": 26, "PageTables": 7, "VmallocUsed": 0,
  "MemFree": 98, "Inactive(file)": 179, "SwapCached": 0,
  "HugePages_Total": 0, "Inactive(anon)": 104, "HugePages_Rsvd": 0,
  "Buffers": 21, "SReclaimable": 39, "Cached": 613,
  "Mlocked": 3, "SwapTotal": 1021, "NFS_Unstable": 0,
  "CommitLimit": 1518, "Hugepagesize": 2, "SwapFree": 1016,
  "WritebackTmp": 0, "Committed_AS": 1410, "AnonHugePages": 130,
  "DirectMap2M": 966, "Unevictable": 3, "HugePages_Surp": 0,
  "Dirty": 3, "HugePages_Free": 0, "MemAvailable": 599,
  "Active(file)": 426, "Slab": 54, "Active": 525,
  "KernelStack": 2, "VmallocChunk": 0, "AnonPages": 177,
  "Bounce": 0, "HardwareCorrupted": 0, "DirectMap4k": 57
]
```

而下面则是一个典型的mac OS X内存摘要，看起来还有 4.5GB 空闲内存：

``` swift
[
  "hits": 0, "faults": 3154324, "cow": 31476,
  "wired": 3576, "reactivations": 366, "zero_filled": 2296248,
  "pageins": 13983, "lookups": 1021, "pageouts": 0,
  "active": 6967, "free": 4455, "inactive": 1008
]

```

### 网络流量

调用静态属性 `SysInfo.Net` 可以获得当前所有网卡流量统计，以字典 `[String:[String:Int]]` 统计，其中条目为网络适配器名称，对应的子项字典中有两个条目： `i` 表示接收字节数，`o` 表示发送字节数，二者单位都是“KB”（千字节）

``` swift

if let net = SysInfo.Net {
  print(net)
}

```


如果调用成功，则会根据操作系统是 mac 还是 linux 不同，输出以下结果:

``` swift
// 典型的 mac OS X 网络统计结果，可以看到其中真正的网卡"en0"收到了1MB数据。
[
	"p2p0": ["o": 0, "i": 0], 
	"stf0": ["o": 0, "i": 0], 
	"vboxnet0": ["o": 0, "i": 1], 
	"gif0": ["o": 0, "i": 0], 
	"lo0": ["o": 0, "i": 887], 
	"bridge0": ["o": 0, "i": 0], 
	"utun0": ["o": 0, "i": 0], 
	"awdl0": ["o": 0, "i": 318], 
	"en1": ["o": 0, "i": 0], 
	"en0": ["o": 0, "i": 1063], 
	"en2": ["o": 0, "i": 0]
]

// 典型的 Linux 网络统计结果，可以看到其中真正的网卡 "enp0s3" 收到了 0.6MB 数据并发出了 506KB 字节。
[
	"virbr0": ["o": 0, "i": 0], 
	"enp0s8": ["o": 506, "i": 614], 
	"virbr0-nic": ["o": 0, "i": 0], 
	"lo": ["o": 1804, "i": 1804], 
	"enp0s3": ["o": 158, "i": 7594]
]
```

### 磁盘读写监控

调用静态方法 `SysInfo.Disk` 可以获得磁盘当前活动统计数据，返回结果为一个 `[String:[String: UInt64]]` 字典，如下所示。关于这些参数指标，请参考操作系统手册。

#### Linux 发行说明

``` swift
print(SysInfo.Disk)

// Linux 典型输出
[
  "sda":
    [
      "io_ms": 7516, "reads_merged": 17, "reads_completed": 14993,
      "writing_ms": 9772, "io_in_progress": 0, "writes_completed": 4921,
      "sectors_read": 1292762, "reading_ms": 11952, "writes_merged": 5738,
      "sectors_written": 969480, "weighte_io_ms": 21636
    ],
  "sda2":
    [
      "io_ms": 0, "reads_merged": 0, "reads_completed": 4,
      "writing_ms": 0, "io_in_progress": 0, "writes_completed": 0,
      "sectors_read": 8, "reading_ms": 0, "writes_merged": 0,
      "sectors_written": 0, "weighte_io_ms": 0
    ],
  "sda1":
    [
      "io_ms": 7252, "reads_merged": 7, "reads_completed": 14817,
      "writing_ms": 9520, "io_in_progress": 0, "writes_completed": 3971,
      "sectors_read": 1281954, "reading_ms": 11908, "writes_merged": 5426,
      "sectors_written": 966696, "weighte_io_ms": 21340
    ]
]

```

#### Mac OS X 发行说明

请注意如果在循环内使用 `SysInfo.Disk`，则强烈建议增加 `autoreleasepool{ }` 闭包，用于避免不必要的内存堆积：

``` swift

autoreleasepool(invoking: {
  let io = SysInfo.Disk
  print(io)
})

// macOS下的参考输出
[
  "disk0":
    [
      "operations_read": 501077, "latency_time_read": 0,
      "bytes_written": 21265645056, "bytes_read": 25022815232,
      "operations_written": 360598, "latency_time_written": 0
    ]
]
```
