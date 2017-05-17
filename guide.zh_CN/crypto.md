# Crypto 加密函数库

Perfect-Crypto 是一个基于OpenSSL的通用加密算法函数库。该函数库为以下加密工作提供高级编程接口：

* 消息摘要码（Digest）和哈希码（Hash）生成
* 加密（Cipher）编码和解码
* 根据指定字节长度生成随机数

同时本函数库还提供相关通用编码函数接口，特别是将二进制数据转化为可打印文本。

为了使用本函数库，请确定在源代码开始段导入`import PerfectCrypto`.

## 函数库初始化

本函数库使用之前需要进行一次性初始化工作，而且需要在程序准备任何实质性编解码之前完成。调用`PerfectCrypto.isInitialized` 会进行初始化操作并在初始化完成后返回真值。虽然允许多次调用，但实际上最好在程序启动过程中使用。

## 类型扩展

本函数库多数函数接口通过扩展基本变量类型完成，分别是：`String`（字符串）、 `Array<UInt8>`（字节数组）、`UnsafeRawBufferPointer`（通用缓冲区连续指针）以及 `UnsafeMutableRawBufferPointer`（通用可变缓冲区连续指针）。对应每个扩展类型增加的函数包括`encode/decode`（编解码）、`encrypt/decrypt`（加密解密）和`digest`（摘要码）。除此之外还包括原始二进制数据的处理、随机数生成等等。

以下是函数参考，按照使用的方便程度进行排序，首先介绍高级调用方法，最后介绍一些基于指针基础上调用时要小心的底层函数。

### 字符串

字符串功能扩展分为两组，第一组是编解码和摘要码生成：

```swift
public extension String {
	/// 根据编码类型将字符串解码为二进制数组
	/// 将字符串的UTF8字符数据进行转换和解码
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// 根据指定的编码方式将字符串编码为一个字节数组。
	/// 将字符串的UTF8字符数据进行转换和编码
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// 根据摘要算法将字符串UTF8内容生成字节数组作为该字符串的摘要码
	func digest(_ digest: Digest) -> [UInt8]?
}
```

`encode` / `decode` 编解码函数使用时需要输入编码类型 `Encoding`，输出结果是字节数组，或者如果在输入数据无效时输出为空。编码类型必须为函数库指定的编码类型枚举，比如 `.hex`（16进制），或者`.base64url`。

以下例子展示了如何将一个字符串进行base64编码，并解码为原字符串：

```swift
let testStr = "Räksmörgåsen"
if let encoded = testStr.encode(.base64),
	let decoded = encoded.decode(.base64),
	let decodedStr = String(validatingUTF8: decoded) {
	print(decodedStr)
	// Räksmörgåsen
}
```

摘要函数 `digest`允许对字符串进行摘要计算，所生成的摘要数据返回结果为字节数组 `[UInt8]`。如果系统不支持对应的摘要算法则返回为空。

下列例子展示了如何对一个字符串进行SHA256摘要计算，输出结果再次转换为16进制可打印格式文本：

```swift
let testStr = "Hello, world!"
if let digestBytes = testStr.digest(.sha256),
	let hexBytes = digestBytes.encode(.hex),
	let hexBytesStr = String(validatingUTF8: hexBytes) {
	print(hexBytesStr)
	// 315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3
}
```

第二组字符串扩展是从C语言字符串，也就是**可变长零结尾字符串**直接创建Swift标准字符串的方法，输入类型可以是 `[UInt8]` 或 `UnsafeRawBufferPointer`。

```swift
public extension String {
	/// 从UTF8数组创建字符串，数组长度决定了转换内容长度；如果数据无效则字符串为空
	init?(validatingUTF8 a: [UInt8])
	/// 从指针构造字符串。指针可以不是零值结尾，而是由缓冲区长度决定转换内容长度
	/// 输入内容无效则返回为空
	init?(validatingUTF8 ptr: UnsafeRawBufferPointer?)
	/// 从字符串内获得缓冲区指针。
	func withBufferPointer<Result>(_ body: (UnsafeRawBufferPointer) throws -> Result) rethrows -> Result
}
```

上述最后一个方法`withBufferPointer`是专门用于获得该字符串对应的`UnsafeRawBufferPointer`缓冲区指针，内容包含的是UTF8数据。缓冲区只能在声明的闭包内使用。

### Array&lt;UInt8&gt;

数组类型扩展只允许 `UInt8` 无符号字节整型。功能上包括编解码、加密解密、摘要计算以及创建随机数。

```swift
public extension Array where Element: Octal {
	/// 将数组转换为指定编码类型的线性表。
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// 将数组解码为制定编码类型的线性表。
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// 摘要计算
	func digest(_ digest: Digest) -> [UInt8]?
	/// 根据指定密码、类型和初始化向量进行加密
	func encrypt(_ cipher: Cipher, key: [UInt8], iv: [UInt8]) -> [UInt8]?
	/// 根据指定密码、类型和初始化向量进行解密
	func decrypt(_ cipher: Cipher, key: [UInt8], iv: [UInt8]) -> [UInt8]?
}
```

数组编解码和摘要计算的方法与字符串的上述扩展是一一对应的，除了数据类型为数组之外并无任何区别。使用时需要输入编码类型，返回结果为`[UInt8]`字节数组；如果输入内容无效则返回为空。编码类型必须为本函数库编码类型枚举所列，比如`.hex`（16进制） 或 `.base64url`。

摘要函数 `digest`允许对数组进行摘要计算，所生成的摘要数据返回结果为字节数组 `[UInt8]`。如果系统不支持对应的摘要算法则返回为空。

加密和解密函数 `encrypt` / `decrypt` 根据输入的加密类型枚举进行加密解密操作，使用时需要输入密码和初始化向量。

根据加密算法的需要，钥匙和初始化向量的长度是不同的。在加密算法枚举中会提供每种算法所需要的 `blockSize` （缓冲区尺寸）、`keyLength` （钥匙长度）和 `ivLength` （初始化向量长度），所有这些长度类型的单位都是字节。

下列代码展示了如何使用`.aes_256_cbc`算法对一个随机数组进行加密。这个例子中的钥匙和初始化向量也都是根据算法要求的对应长度自动生成的随机数。

```swift
let cipher = Cipher.aes_256_cbc
	// 待加密数据
let random = [UInt8](randomCount: 2048)
	// 加密/解密使用的密码
let key = [UInt8](randomCount: cipher.keyLength)
	// 初始化向量
let iv = [UInt8](randomCount: cipher.ivLength)

if let encrypted = random.encrypt(cipher, key: key, iv: iv),
	let decrypted = encrypted.decrypt(cipher, key: key, iv: iv) else {
	
	for (a, b) in zip(decrypted, random) {
		(a == b)
	}
}
```

其中，随机数组的创建使用了下列扩展类型：

```swift
public extension Array where Element: Octal {
	/// 根据指定长度创建数组并随机填写内容。
	init(randomCount count: Int)
}
```

随机数组意味着该数组的每一个字节元素的内容都是随机数。下列例子展示了如何生成一个16字节随机数并转换为base64字符串。

```swift
	// 生成16字节长度的随机数
let random = [UInt8](randomCount: 16)
if let base64 = random.encode(.base64),
	let base64Str = String(validatingUTF8: base64) {
	print(base64Str)
}
```

### 指针类型扩展

下列内容为在指针 `UnsafeMutableRawBufferPointer` 和 `UnsafeRawBufferPointer` 基础上使用缓冲区扩展而来的各类操作。其实功能和上面的内容都是一致的，只不过使用指针效率会更高一些。

### 可变缓冲区指针

使用可变缓冲区指针生成随机数组：

```swift
public extension UnsafeMutableRawBufferPointer {
	/// 根据指定长度分配内存缓冲区并自动填写随机数
	static func allocateRandom(count size: Int) -> UnsafeMutableRawBufferPointer?
	/// 用随机数初始化缓冲区
	func initializeRandom()
}
```

静态方法 `allocateRandom` 用于分配一段内存缓冲区并自动填写随机数。使用后必须手工调用`deallocate`注销并释放该内存段。

而`initializeRandom`方法则是假设内存已经分配好了，并使用这个已经分配好的指针进行随机内容填写，填写长度由该缓冲区指针长度 `.count` 值决定。

### 不可变缓冲区指针

下列扩展同样提供与上述`allocateRandom`相同的静态方法，以及加密解密、编码解码或者摘要计算。

```swift
public extension UnsafeRawBufferPointer {
	/// 分配指定长度内存并填写随机数
	static func allocateRandom(count size: Int) -> UnsafeRawBufferPointer?
	/// 使用缓冲区生成编码内容，返回结果使用完后必须自行释放
	func encode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// 使用缓冲区生成解码内容，返回结果使用完后必须自行释放
	func decode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// 生成摘要内容，生成结果必须手工释放
	func digest(_ digest: Digest) -> UnsafeMutableRawBufferPointer?
	/// 使用指定算法、钥匙和初始化向量进行加密，返回结果必须手工释放
	func encrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
	/// 使用指定算法、钥匙和初始化向量进行解密，返回结果必须手工释放
	func decrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
}
```

使用上述指针时要格外小心，所有以 `UnsafeMutableRawBufferPointer` 为返回类型的结果必须由调用者进行手工内存释放。而且每个缓冲指针都有自己的 `.count` 属性用于计量其内存长度。而`UnsafeRawBufferPointer` 指针则不需要手工释放内存，但是也不能修改具体内容。

## 算法

本函数库提供了以下类型的编解码算法、加密解密算法和摘要算法：

* 编码： base64, base64url, hex
* 摘要： md4, md5, sha, sha1, dss, dss1, ecdsa, sha224, sha256, sha384, sha512, ripemd160, whirlpool, custom(String)
* 加密： des\_ecb, des\_ede, des\_ede3, des\_ede\_ecb, des\_ede3\_ecb, des\_cfb64, des\_cfb1, des\_cfb8, des\_ede\_cfb64, des\_ede3\_cfb1, des\_ede3\_cfb8, des\_ofb, des\_ede\_ofb, des\_ede3\_ofb, des\_cbc, des\_ede\_cbc, des\_ede3\_cbc, desx\_cbc, des\_ede3\_wrap, rc4, rc4\_40, rc4\_hmac\_md5, rc2\_ecb, rc2\_cbc, rc2\_40\_cbc, rc2\_64\_cbc, rc2\_cfb64, rc2\_ofb, bf\_ecb, bf\_cbc, bf\_cfb64, bf\_ofb, cast5\_ecb, cast5\_cbc, cast5\_cfb64, cast5\_ofb, aes\_128\_ecb, aes\_128\_cbc, aes\_128\_cfb1, aes\_128\_cfb8, aes\_128\_cfb128, aes\_128\_ofb, aes\_128\_ctr, aes\_128\_ccm, aes\_128\_gcm, aes\_128\_xts, aes\_128\_wrap, aes\_192\_ecb, aes\_192\_cbc, aes\_192\_cfb1, aes\_192\_cfb8, aes\_192\_cfb128, aes\_192\_ofb, aes\_192\_ctr, aes\_192\_ccm, aes\_192\_gcm, aes\_192\_wrap, aes\_256\_ecb, aes\_256\_cbc, aes\_256\_cfb1, aes\_256\_cfb8, aes\_256\_cfb128, aes\_256\_ofb, aes\_256\_ctr, aes\_256\_ccm, aes\_256\_gcm, aes\_256\_xts, aes\_256\_wrap, aes\_128\_cbc\_hmac\_sha1, aes\_256\_cbc\_hmac\_sha1, aes\_128\_cbc\_hmac\_sha256, aes\_256\_cbc\_hmac\_sha256, camellia\_128\_ecb, camellia\_128\_cbc, camellia\_128\_cfb1, camellia\_128\_cfb8, camellia\_128\_cfb128, camellia\_128\_ofb, camellia\_192\_ecb, camellia\_192\_cbc, camellia\_192\_cfb1, camellia\_192\_cfb8, camellia\_192\_cfb128, camellia\_192\_ofb, camellia\_256\_ecb, camellia\_256\_cbc, camellia\_256\_cfb1, camellia\_256\_cfb8, camellia\_256\_cfb128, camellia\_256\_ofb, seed\_ecb, seed\_cbc, seed\_cfb128, seed\_ofb, custom(String)

其中，摘要算法和加密算法的枚举类型还提供了定制的方法，即`custom`类型，由用户输入的算法名称决定，用于用户自行提供算法方案。所有算法基本上都基于OpenSSL但少数内容则是直接用Swift语言实现的，比如hex十六进制类型。

