# Bytes

The Bytes object provides simple streaming of common Swift values to and from a UInt8 array. It supports importing and exporting UInt8, UInt16, UInt32 and UInt64 values. When importing these values, they are appended to the end of the contained array. When exporting, a repositionable position marker is kept indicating the current export location.

The primary purpose behind the Bytes object is to enable binary network payloads to be easily assembled and decomposed. The resulting UInt8 array is available through the ```data``` property.

The following example illustrates importing values of various sizes and then exporting and validating the values. It also shows how to reposition the marker and how to determine how many bytes remain for export.

```swift
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

