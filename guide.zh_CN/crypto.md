# Crypto 加密函数库

Perfect-Crypto 是一个基于OpenSSL的通用加密算法函数库。该函数库为以下加密工作提供高级编程接口：

* 消息摘要码（Digest）和哈希码（Hash）生成
* 加密（Cipher）编码和解码
* 根据指定字节长度生成随机数

同时本函数库还提供相关通用编码函数接口，特别是将二进制数据转化为可打印文本。

为了使用本函数库，请确定在源代码开始段导入`import PerfectCrypto`.

## 编译

请在您的Package.swift文件中增加下列依存关系：

```
.Package(url: "https://github.com/PerfectlySoft/Perfect-Crypto.git", majorVersion: 3)
```

## Linux 编译说明

请确保您的系统上已经安装了 libssl-dev 函数库

```
sudo apt-get install libssl-dev
```

## 概述

本函数库将OpenSSL的部分功能进行了封装并在Swift基本类型上进行了扩展，主要内容包括：

* 对于字符串、[UInt8] 和 UnsafeRawBufferPointer 指针增加了基本的编解码、摘要码和加密操作。
* 针对于非零结尾指针创建UTF-8字符串的方法
* 对OpenSSL BIO函数类的封装，提供可过滤的链式操作。

## 使用范例

### 16进制编解码

```swift
let testStr = "Hello, world!"
guard let hexBytes = testStr.encode(.hex) else {
	return
}

String(validatingUTF8: hexBytes) == "48656c6c6f2c20776f726c6421"

guard let unHex = hexBytes.decode(.hex) else {
	return
}

String(validatingUTF8: unHex) == testStr

```

### Base 64 编解码

```swift
let testStr = "Hello, world!"
guard let baseBytes = testStr.encode(.base64) else {
	return
}

String(validatingUTF8: baseBytes) == "SGVsbG8sIHdvcmxkIQ=="

guard let unBase = baseBytes.decode(.base64) else {
	return
}

String(validatingUTF8: unBase) == testStr
```

### 摘要码

```swift
let testStr = "Hello, world!"
let testAnswer = "315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3"
guard let enc = testStr.digest(.sha256)?.encode(.hex) else {
	return
}

String(validatingUTF8: enc) == testAnswer
```

### HMAC 签名和校验

下列代码用于 HMAC-SHA1 内容签名和 base64 编码，然后解码并校验。请根据需要自行调整.sha1 或者 .base64 算法：

```swift
let password = "用于测试的密码"
let data = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
	
if let signed = data.sign(.sha1, key: HMACKey(password))?.encode(.base64),
	let base64Str = String(validatingUTF8: signed),
	
	let reRawData = base64Str.decode(.base64) {
	
	let verifyResult = data.verify(.sha1, signature: reRawData, key: HMACKey(password))
	XCTAssert(verifyResult)
} else {
	XCTAssert(false, "签名失败")
}
```

### API参考

``` swift
public extension String {
	/// 从UTF8数组创建字符串，数组长度决定了转换内容长度；如果数据无效则字符串为空
	init?(validatingUTF8 a: [UInt8])
	/// 从指针构造字符串。指针可以不是零值结尾，而是由缓冲区长度决定转换内容长度
	/// 输入内容无效则返回为空
	init?(validatingUTF8 ptr: UnsafeRawBufferPointer?)
	/// 从字符串内获得缓冲区指针。
	func withBufferPointer<Result>(_ body: (UnsafeRawBufferPointer) throws -> Result) rethrows -> Result
}

public extension String {
	/// 将字符串转换为指定编码类型的线性表。
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// 将字符串解码为制定编码类型的线性表。
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// 摘要计算
	func digest(_ digest: Digest) -> [UInt8]?
	/// 根据算法和钥匙签署字符串并生成字节数组
	func sign(_ digest: Digest, key: Key) -> [UInt8]?
	/// 根据字符串验证签名
	/// 验证成功返回真，否则返回伪
	func verify(_ digest: Digest, signature: [UInt8], key: Key) -> Bool
	/// 根据缓冲区密文、密码和盐对数据进行加密
	/// 字符串的 UTF8 字符将被编码
	/// 返回数据为CMS格式的PEM编码。
	func encrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
	/// 根据密码和盐进行CMS格式PEM编码数据解码
	/// 解码结果必须为UTF8编码否则操作失败
	func decrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
}

public protocol Octal {}
extension UInt8: Octal {}

public extension Array where Element: Octal {
	/// 将数组转换为指定编码类型的线性表。
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// 将数组解码为制定编码类型的线性表。
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// 摘要计算
	func digest(_ digest: Digest) -> [UInt8]?
	/// 根据算法和钥匙签署字符串并生成字节数组
	func sign(_ digest: Digest, key: Key) -> [UInt8]?
	/// 根据字符串验证签名
	/// 验证成功返回真，否则返回伪
	func verify(_ digest: Digest, signature: [UInt8], key: Key) -> Bool
	/// 根据缓冲区密文、密码和盐对数据进行加密
	/// 字符串的 UTF8 字符将被编码
	/// 返回数据为CMS格式的PEM编码。
	func encrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
	/// 根据密码和盐进行CMS格式PEM编码数据解码
	/// 解码结果必须为UTF8编码否则操作失败
	func decrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
}

public extension UnsafeRawBufferPointer {
	/// 使用缓冲区生成编码内容，返回结果使用完后必须自行释放
	func encode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// 使用缓冲区生成解码内容，返回结果使用完后必须自行释放
	func decode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// 生成摘要内容，生成结果必须手工释放
	func digest(_ digest: Digest) -> UnsafeMutableRawBufferPointer?
	/// 根据算法和钥匙签署字符串并生成字节数组
	/// 返回结果必须由用户自行释放内存
	func sign(_ digest: Digest, key: Key) -> UnsafeMutableRawBufferPointer?
	/// 根据数据验证签名
	/// 验证成功返回真，否则返回伪
	func verify(_ digest: Digest, signature: UnsafeRawBufferPointer, key: Key) -> Bool
	/// 根据密文、密码和iv（初始化向量）对数据进行加密
	/// 生成结果必须手工释放
	func encrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
	/// 根据密文、密码和iv（初始化向量）对数据进行解密
	/// 生成结果必须手工释放
	func decrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
	/// 根据密文、密码和盐对数据进行进行CMS格式PEM加密
	/// 生成结果必须手工释放
	func encrypt(_ cipher: Cipher,
	             password: UnsafeRawBufferPointer,
	             salt: UnsafeRawBufferPointer,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> UnsafeMutableRawBufferPointer?
   /// 根据密码和盐对数据进行进行CMS格式PEM解密
	/// 生成结果必须手工释放
	func decrypt(_ cipher: Cipher,
	             password: UnsafeRawBufferPointer,
	             salt: UnsafeRawBufferPointer,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> UnsafeMutableRawBufferPointer?
}

public extension UnsafeRawBufferPointer {
	/// 根据长度要求填充随机数
	///
	/// - 结果：内存被分配并被自动初始化为随机数
	static func allocateRandom(count size: Int) -> UnsafeRawBufferPointer? 
}
```

### JSON 网络通行证 (JWT)

本组件为JWT创建和验证函数库。

JSON Web Token (以下简称网络通行证JWT) 为开放互联网标准协议 (RFC 7519) 用于定义在通信双方会话过程中以JSON为载体安全传输加密信息的方法。该信息可以用于互信和校验因为采用数字签名。JWTs 可用于HMAC算法加密签名，或者采用RSA公开/私有钥匙对签名方法，详见 [JWT](https://jwt.io/introduction/).

首先，新的JWT令牌可以可用 `JWTCreator` 对象创建。

```swift
/// 创建并签署新的 JWT 令牌。
public struct JWTCreator {
	/// 根据荷载内容创建新的通行券。
	/// 荷载内容可以用于创建JWT字符串
	public init?(payload: [String:Any])
	/// 使用HMAC钥匙创建并返回一个新的JWT令牌。
	/// 可根据需要自行追加其他头数据
	/// 如果生成令牌中出现问题，会抛出签名错误 JWT.Error.signingError
	public func sign(alg: JWT.Alg, key: String, headers: [String:Any] = [:]) throws -> String
	/// 根据指定钥匙签署并生成新的 JWT 令牌。
	/// 可根据需要自行追加其他头数据
	/// 钥匙类型必须与算法 `algo` 兼容
	/// 如果生成令牌中出现问题，会抛出签名错误 JWT.Error.signingError
	public func sign(alg: JWT.Alg, key: Key, headers: [String:Any] = [:]) throws -> String
}
```

现有 JWT 通行证可以通过 `JWTVerifier` 对象进行验证

```swift
/// 接受一个 JWT 通行证并验证签名
public struct JWTVerifier {
	/// 从通行证内取出的头数据
	public var header: [String:Any]
	/// 通行证内的荷载数据
	public var payload: [String:Any]
	/// 从通行证字符串中创建 JWTVerifier 签名对象。通行证格式应该为 "aaaa.bbbb.cccc" 
	/// 如果通行证无效则返回为 nil
	/// *注意这一步不做验证* 必须手工调用 `verify` 验证钥匙
	/// 如果验证成功，则头数据`.headers`和荷载数据 `.payload`才能安全生效
	public init?(_ jwt: String)
	/// 使用指定算法和HMAC钥匙验证通行证
	/// 如果生成令牌中出现问题，会抛出验证错误 JWT.Error.verificationError
	/// 如果验证无误则正常执行
	/// 参数 `algo` 必须与通行证中的 "alg" 头数据字段吻合
	public func verify(algo: JWT.Alg, key: String) throws
	/// 使用指定算法和HMAC钥匙验证通行证
	/// 如果生成令牌中出现问题，会抛出验证错误 JWT.Error.verificationError
	/// 如果验证无误则正常执行
	/// 参数 `algo` 必须与通行证中的 "alg" 头数据字段吻合
	public func verify(algo: JWT.Alg, key: Key) throws
}
```

以下示范说明了如何创建并使用“HS256”算法验证一个网络通行证。

```swift
let name = "John Doe"
let tstPayload = ["sub": "1234567890", "name": name, "admin": true] as [String : Any]
let secret = "secret"
guard let jwt1 = JWTCreator(payload: tstPayload) else {
	return // fatal error
}
let token = try jwt1.sign(alg: .hs256, key: secret)
guard let jwt = JWTVerifier(token) else {
  return // fatal error
}
try jwt.verify(algo: .hs256, key: HMACKey(secret))
let fndName = jwt.payload["name"] as? String
// name == fndName!
```

⚠️注意⚠️ JWTVerifier 能够验证通行证加密，但是 ⚠️**不会**⚠️ 验证数据内容，比如签发者（iss）和有效期（exp）。用户需要自行从荷载数据字典中提取上述信息并进行进一步用户身份验证。



### 算法清单

```swift
/// 编码方法
public enum Encoding {
	case base64
	case hex
}

/// 摘要码算法
public enum Digest {
	case md4
	case md5
	case sha
	case sha1
	case dss
	case dss1
	case ecdsa
	case sha224
	case sha256
	case sha384
	case sha512
	case ripemd160
	case whirlpool
	
	case custom(String)
}

/// 可用密文
public enum Cipher {
	case des_ecb
	case des_ede
	case des_ede3
	case des_ede_ecb
	case des_ede3_ecb
	case des_cfb64
	case des_cfb1
	case des_cfb8
	case des_ede_cfb64
	case des_ede3_cfb1
	case des_ede3_cfb8
	case des_ofb
	case des_ede_ofb
	case des_ede3_ofb
	case des_cbc
	case des_ede_cbc
	case des_ede3_cbc
	case desx_cbc
	case des_ede3_wrap
	case rc4
	case rc4_40
	case rc4_hmac_md5
	case rc2_ecb
	case rc2_cbc
	case rc2_40_cbc
	case rc2_64_cbc
	case rc2_cfb64
	case rc2_ofb
	case bf_ecb
	case bf_cbc
	case bf_cfb64
	case bf_ofb
	case cast5_ecb
	case cast5_cbc
	case cast5_cfb64
	case cast5_ofb
	case aes_128_ecb
	case aes_128_cbc
	case aes_128_cfb1
	case aes_128_cfb8
	case aes_128_cfb128
	case aes_128_ofb
	case aes_128_ctr
	case aes_128_ccm
	case aes_128_gcm
	case aes_128_xts
	case aes_128_wrap
	case aes_192_ecb
	case aes_192_cbc
	case aes_192_cfb1
	case aes_192_cfb8
	case aes_192_cfb128
	case aes_192_ofb
	case aes_192_ctr
	case aes_192_ccm
	case aes_192_gcm
	case aes_192_wrap
	case aes_256_ecb
	case aes_256_cbc
	case aes_256_cfb1
	case aes_256_cfb8
	case aes_256_cfb128
	case aes_256_ofb
	case aes_256_ctr
	case aes_256_ccm
	case aes_256_gcm
	case aes_256_xts
	case aes_256_wrap
	case aes_128_cbc_hmac_sha1
	case aes_256_cbc_hmac_sha1
	case aes_128_cbc_hmac_sha256
	case aes_256_cbc_hmac_sha256
	case camellia_128_ecb
	case camellia_128_cbc
	case camellia_128_cfb1
	case camellia_128_cfb8
	case camellia_128_cfb128
	case camellia_128_ofb
	case camellia_192_ecb
	case camellia_192_cbc
	case camellia_192_cfb1
	case camellia_192_cfb8
	case camellia_192_cfb128
	case camellia_192_ofb
	case camellia_256_ecb
	case camellia_256_cbc
	case camellia_256_cfb1
	case camellia_256_cfb8
	case camellia_256_cfb128
	case camellia_256_ofb
	case seed_ecb
	case seed_cbc
	case seed_cfb128
	case seed_ofb
	
	case custom(String)
}
```
