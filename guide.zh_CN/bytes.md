# 字节流转换

Bytes二进制字节流对象是在一个无符号8位整型数组基础上为实现Swift简单流操作而开发的类。该对象支持对所有无符号整型值的互相转换：UInt8、UInt16、UInt32和UInt64。在转换这些数值的流操作过程中，新增内容会在容器数组末端进行追加操作。在导出的流操作过程中，会有一个标明当前数据流位置信息的指示器用于检查进度。该对象是PerfectLib的一个组成部分，如果需要使用Bytes对象，则请在程序开通采用`import PerfectLib`声明库函数调用

该二进制字节流对象的主要设计目的是用于二进制数据在网络上传输过程中的打包和解包（备注：互联网传输协议与本地计算机协议直接存在高低位字节顺序转换问题——译者注）。用于表达结果的8位无符号数组可以通过访问`data`属性实现。

以下例子说明了不同大小不同内容的流数据导入导出以及验证的过程，同时也说明了流位置指示器是如何说明在导出流还有多少字节待操作。

``` swift
let i8 = 254 as UInt8
let i16 = 54045 as UInt16
let i32 = 4160745471 as UInt32
let i64 = 17293541094125989887 as UInt64

let bytes = Bytes()

bytes.import64Bits(from: i64)
	.import32Bits(from: i32)
	.import16Bits(from: i16)
	.import8Bits(from: i8)

let bytes2 = Bytes()
bytes2.importBytes(from: bytes)

XCTAssert(i64 == bytes2.export64Bits())
XCTAssert(i32 == bytes2.export32Bits())
XCTAssert(i16 == bytes2.export16Bits())
bytes2.position -= sizeof(UInt16.self)
XCTAssert(i16 == bytes2.export16Bits())
XCTAssert(bytes2.availableExportBytes == 1)
XCTAssert(i8 == bytes2.export8Bits())
```
