# Perfect SysInfo 

This project provides a Swift library to monitor system performance.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project but can also be used as an independent module.

Ensure you have installed and activated the latest Swift 4.0 tool chain.

## Quick Start

Add Perfect SysInfo library to your Package.swift:

``` swift

.Package(url: "https://github.com/PerfectlySoft/Perfect-SysInfo.git", majorVersion: 3)

```

Add library header to your source code:

``` swift

import PerfectSysInfo

```

Now SysInfo class is available to call.

### CPU Usage

Call static variable `SysInfo.CPU` will return a dictionary `[String: [String: Int]]` of all CPU usage, for example:

``` swift

print(SysInfo.CPU)

//here is a typical return of single CPU (from Linux): 

[
  "cpu0":
    ["nice": 1201, "system": 3598, "user": 8432, "idle": 8657606],
  "cpu":
    ["nice": 1201, "system": 3598, "user": 8432, "idle": 8657606]
]

// and the following is another example with 8 cores (from Mac):
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

The record is a structure of `N+1` entries, where `N` is the number of CPU and `1` is the summary, so the each record will be tagged with "cpu0" ... "cpuN-1" and the tag "cpu" represents the average of overall. Each entries will contain `idle`, `user`, `system` and `nice` to represent the cpu usage time. In a common sense, `idle` shall be as large as possible to indicate the CPU is not busy.

### Memory Usage

Call static property `SysInfo.Memory` will return a dictionary `[String: Int]` with memory metrics in ** MB **:

``` swift

print(SysInfo.Memory)

```

** Note ** Since system information is subject to operating system type, so please use directive `#if os(Linux) #else #endif` determine OS type before reading system metrics; The definition of each counter is out of the scope of this document, please see OS manual for detail.

Typical Linux memory looks like this ( 1G total memory with about 599MB available):

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

And here is a typical mac OS X memory summary, which indicates that there is about 4.5GB free memory:

``` swift
[
  "hits": 0, "faults": 3154324, "cow": 31476,
  "wired": 3576, "reactivations": 366, "zero_filled": 2296248,
  "pageins": 13983, "lookups": 1021, "pageouts": 0,
  "active": 6967, "free": 4455, "inactive": 1008
]

```

### Network Traffic

Call static property `SysInfo.Net` will return total traffic summary from all interfaces as a dictionary `[String: [String: Int]]` where the key represents the network interface name, and the value is a detailed dictionary with two key-value pairs - `i` stands for receiving and `o` for transmitting, both in KB:

``` swift

if let net = SysInfo.Net {
  print(net)
}

```

If success, it will print something like these mac  / linux outputs:

``` swift
// typical mac os x network summary, where the only physical network
// adapter "en0" has 1MB incoming data totally.

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

// typical linux network summary, where the only physical network
// adapter "enp0s3" has received 0.6MB data and sent out 506KB in the same time.

[
	"virbr0": ["o": 0, "i": 0], 
	"enp0s8": ["o": 506, "i": 614], 
	"virbr0-nic": ["o": 0, "i": 0], 
	"lo": ["o": 1804, "i": 1804], 
	"enp0s3": ["o": 158, "i": 7594]
]
```

### Disk IO

Call static method `SysInfo.Disk` may inspect disk i/o activity statistics in real time. It will return a `[String:[String: UInt64]]` dictionary with metrics as sample below. For more information of these counters, please refer to the operating system manual.

#### Linux Release Notes

``` swift
print(SysInfo.Disk)

// here is a sample output from Linux:
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

#### Mac OS X Release Notes

Please note that if using `SysInfo.Disk` in a loop, then an `autoreleasepool{ }` is strongly recommend to avoid unnecessary memory caching on such objects:

``` swift

autoreleasepool(invoking: {
  let io = SysInfo.Disk
  print(io)
})

// here is the sample output from macOS:
[
  "disk0":
    [
      "operations_read": 501077, "latency_time_read": 0,
      "bytes_written": 21265645056, "bytes_read": 25022815232,
      "operations_written": 360598, "latency_time_written": 0
    ]
]
```
