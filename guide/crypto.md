# Crypto

The Perfect-Crypto package is a general purpose cryptography library built on OpenSSL. It provides high level objects for dealing with the following cryptographic tasks:

* Message digests and hashes, sign/verify
* Cipher based encryption/decryption
* Random data generation of arbitrary byte lengths
* HMAC key generation
* PEM format public/private key reading
* JWT (JSON Web Token) creation and validation

It also provides some encoding related functions which are generally useful but are commonly used along with cryptography, particularly when converting binary data to and from a character printable state.

To use the functionality of this package ensure you `import PerfectCrypto`.

## Initialization

The underlying cryptography library requires a one-time initialization to be performed before any related functions are used. This is generally done once when your program starts before performing any actual tasks. Calling `PerfectCrypto.isInitialized` will initialize the library and return true. Calling this more than once is fine, however one would generally only call this once when boot-strapping your application.

## Extensions

Much of the functionality provided by this package is through extensions on several of the Swift builtin types, namely: `String`, `Array<UInt8>`, `UnsafeRawBufferPointer`, and `UnsafeMutableRawBufferPointer`. The extensions mainly consist of adding `encode/decode`, `encrypt/decrypt`, `sign/verify` and `digest` functions for these types. Others provide convenience functions for dealing with raw binary data, generating random data, etc.

The extensions are listed below in order of "convenience". The higher level functions are listed first and the more efficient "unsafes" are listed at the end.

### String

The String extensions are divided into two groups. The first group provides the encode, decode and digest functions for Strings.

```swift
public extension String {
	/// Decode the String into an array of bytes using the indicated encoding.
	/// The string's UTF8 characters are decoded.
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// Encode the String into an array of bytes using the indicated encoding.
	/// The string's UTF8 characters are encoded.
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// Perform the digest algorithm on the String's UTF8 bytes.
	func digest(_ digest: Digest) -> [UInt8]?
	/// Sign the String data into an array of bytes using the indicated algorithm and key.
	func sign(_ digest: Digest, key: Key) -> [UInt8]?
	/// Verify the signature against the String data.
	/// Returns true if the signature is verified. Returns false otherwise.
	func verify(_ digest: Digest, signature: [UInt8], key: Key) -> Bool
	/// Encrypt this buffer using the indicated cipher, password, and salt.
	/// The string's UTF8 characters are encoded.
	/// Resulting data is in PEM encoded CMS format.
	func encrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
	/// Decrypt this PEM encoded CMS buffer using the indicated password and salt.
	/// Resulting decrypted data must be valid UTF-8 characters or the operation will fail.
	func decrypt(_ cipher: Cipher,
	             password: String,
	             salt: String,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> String?
}
```

The `encode` and `decode` functions are given an `Encoding` type and will return the result as a `[UInt8]` or nil if the input data was invalid for that particular encoding. The encoding type must be one of the valid `Encoding` enums. For example `.hex`, or `.base64url`. 

This example shows how one would encode a String as base64 and decode it back into the original.

```swift
let testStr = "Räksmörgåsen"
if let encoded = testStr.encode(.base64),
	let decoded = encoded.decode(.base64),
	let decodedStr = String(validatingUTF8: decoded) {
	print(decodedStr)
	// Räksmörgåsen
}
```

The `digest` function allows one to perform a message digest operation on the String's characters. The digest data is returned as a `[UInt8]`. nil is returned if the indicated encoding is not supported by the underlying system.

This example shows a sha256 digest calculated for a String. The resulting value is then converted into a printable hexadecimal String.

```swift
let testStr = "Hello, world!"
if let digestBytes = testStr.digest(.sha256),
	let hexBytes = digestBytes.encode(.hex),
	let hexBytesStr = String(validatingUTF8: hexBytes) {
	print(hexBytesStr)
	// 315f5bdb76d078c43b8ac0064e4a0164612b1fce77c869345bfc94c75894edd3
}
```

The `sign` and `verify` functions permit a block of data to be cryptographically signed using a specified key and then later verified to ensure that the data has not been modified in any way. Signing can be done with either HMAC, RSA or EC type keys.

The `encrypt` and `decrypt` functions accept a Cipher, password and salt value and encrypt/decrypt the data. These encryption functions produce and consume PEM encoded Cryptographic Message Syntax (CMS) data.

```swift
let cipher = Cipher.aes_256_cbc
let password = "this is a good pw"
let salt = "this is a salty salt"
let data = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
guard let result = data.encrypt(cipher, password: password, salt: salt) else {
	return // fatal error
}
guard let decryptedData = result.decrypt(cipher, password: password, salt: salt) else {
	return // decryption error
}
// decryptedData == data
```

The second group of String extensions add convenience functions for creating a String from *non null terminated* UTF-8 characters. These characters can be given through either a `[UInt8]` or `UnsafeRawBufferPointer`.

```swift
public extension String {
	/// Construct a string from a UTF8 character array.
	/// The array's count indicates how many characters are to be converted.
	/// Returns nil if the data is invalid.
	init?(validatingUTF8 a: [UInt8])
	/// Construct a string from a UTF8 character pointer.
	/// Character data does not need to be null terminated.
	/// The buffer's count indicates how many characters are to be converted.
	/// Returns nil if the data is invalid.
	init?(validatingUTF8 ptr: UnsafeRawBufferPointer?)
	/// Obtain a buffer pointer for the String's UTF8 characters.
	func withBufferPointer<Result>(_ body: (UnsafeRawBufferPointer) throws -> Result) rethrows -> Result
}
```

The final function, `withBufferPointer`, is useful for obtaining an `UnsafeRawBufferPointer` containing the String's UTF8 characters. The buffer is valid only within the provided closure/function's body.

### Array&lt;UInt8&gt;

The extensions on Array are only provided for those containing `UInt8` values. These functions provide encode/decode, encrypt/decrypt and digest operations as well as an initializer which allows creating an array of randomly generated data.

```swift
public extension Array where Element: Octal {
	/// Encode the Array into An array of bytes using the indicated encoding.
	func encode(_ encoding: Encoding) -> [UInt8]?
	/// Decode the Array into an array of bytes using the indicated encoding.
	func decode(_ encoding: Encoding) -> [UInt8]?
	/// Digest the Array data into an array of bytes using the indicated algorithm.
	func digest(_ digest: Digest) -> [UInt8]?
	/// Sign the Array data into an array of bytes using the indicated algorithm and key.
	func sign(_ digest: Digest, key: Key) -> [UInt8]?
	/// Verify the array against the signature.
	/// Returns true if the signature is verified. Returns false otherwise.
	func verify(_ digest: Digest, signature: [UInt8], key: Key) -> Bool
	/// Decrypt this buffer using the indicated cipher, key an iv (initialization vector).
	func encrypt(_ cipher: Cipher, key: [UInt8], iv: [UInt8]) -> [UInt8]?
	/// Encrypt this buffer using the indicated cipher, key an iv (initialization vector).
	func decrypt(_ cipher: Cipher, key: [UInt8], iv: [UInt8]) -> [UInt8]?
	/// Encrypt this buffer using the indicated cipher, password, and salt.
	/// Resulting data is PEM encoded CMS format.
	func encrypt(_ cipher: Cipher,
	             password: [UInt8],
	             salt: [UInt8],
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> [UInt8]?
	/// Decrypt this PEM encoded CMS buffer using the indicated password and salt.
	func decrypt(_ cipher: Cipher,
	             password: [UInt8],
	             salt: [UInt8],
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> [UInt8]?
}
```

The encode, decode and digest functions work identically to the String versions, except that the input data is a `[UInt8]`. The encode and decode functions are given an `Encoding` type and will return the result as a `[UInt8]` or nil if the input data was invalid for that particular encoding. The encoding type must be one of the valid `Encoding` enums. For example `.hex`, or `.base64url`.

The `digest` function allows one to perform a message digest operation on the array values. The digest data is returned as a `[UInt8]`. nil is returned if the indicated digest is not supported by the underlying system.

The first set of `encrypt` and `decrypt` functions will encrypt/decrypt the array data based on the indicated `Cipher` enum value. These operations also require input for the key and initialization vector (iv) parameters.

The second set of `encrypt` and `decrypt` functions accept a Cipher, password and salt value and encrypt/decrypt the data. These encryption functions produce and consume PEM encoded Cryptographic Message Syntax (CMS) data.

The sizes of the key and iv arrays will differ based the cipher in use. `Cipher` enum values provide properties for the individual cipher's `blockSize`, `keyLength` and `ivLength`. All of these values are indicated in bytes.

The snippet below will use the `.aes_256_cbc` cipher to encrypt an array of random bytes. The key and the iv for this example are also generated at random based on the sizes required for the cipher.

```swift
let cipher = Cipher.aes_256_cbc
	// the data which will be encrypted
let random = [UInt8](randomCount: 2048)
	// The key value for the encrypt/decrypt
let key = [UInt8](randomCount: cipher.keyLength)
	// Initialization vector
let iv = [UInt8](randomCount: cipher.ivLength)

if let encrypted = random.encrypt(cipher, key: key, iv: iv),
	let decrypted = encrypted.decrypt(cipher, key: key, iv: iv) else {
	
	for (a, b) in zip(decrypted, random) {
		(a == b)
	}
}
```

Arrays containing randomly generated bytes can be produced with the following extension.

```swift
public extension Array where Element: Octal {
	/// Creates a new array containing the specified number of a single random values.
	init(randomCount count: Int)
}
```

Each byte in the resulting array will be generated randomly. The example below generates 16 bytes of random data and converts it to a printable base64 string.

```swift
	// generate 16 random bytes of data
let random = [UInt8](randomCount: 16)
if let base64 = random.encode(.base64),
	let base64Str = String(validatingUTF8: base64) {
	print(base64Str)
}
```

### UnsafeXX Extensions

The extensions provided on `UnsafeMutableRawBufferPointer` and `UnsafeRawBufferPointer` give lower level access to the buffers used for the crypto operations. They provide more efficient behaviour but all the same operations as the Array extension counterparts.

### UnsafeMutableRawBufferPointer

The extensions on `UnsafeMutableRawBufferPointer` permit one to generate or fill a buffer of randomly generated data.

```swift
public extension UnsafeMutableRawBufferPointer {
	/// Allocate memory for `size` bytes with word alignment from the encryption library's
	///	random number generator.
	static func allocateRandom(count size: Int) -> UnsafeMutableRawBufferPointer?
	/// Initialize the buffer with random bytes.
	func initializeRandom()
}
```

The static `allocateRandom` function will return a newly allocated buffer containing the randomized data. You must `deallocate` this buffer just as if you had called the standard `allocate` function.

The `initializeRandom` function will fill the buffer (which is assumed to have been allocated through some other means) with randomized data, up to the buffer's `.count` value.

### UnsafeRawBufferPointer

These extensions provide the same `allocateRandom` static function as above, as well as all of the encode/decode, encrypt/decrypt and digest functions.

```swift
public extension UnsafeRawBufferPointer {
	/// Allocate memory for `size` bytes with word alignment from the encryption library's
	///	random number generator.
	static func allocateRandom(count size: Int) -> UnsafeRawBufferPointer?
	/// Encode the buffer using the indicated encoding.
	/// The return value must be deallocated by the caller.
	func encode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// Decode the buffer using the indicated encoding.
	/// The return value must be deallocated by the caller.
	func decode(_ encoding: Encoding) -> UnsafeMutableRawBufferPointer?
	/// Digest the buffer using the indicated algorithm.
	/// The return value must be deallocated by the caller.
	func digest(_ digest: Digest) -> UnsafeMutableRawBufferPointer?
	/// Sign the buffer using the indicated algorithm and key.
	/// The return value must be deallocated by the caller.
	func sign(_ digest: Digest, key: Key) -> UnsafeMutableRawBufferPointer?
	/// Verify the signature against the buffer.
	/// Returns true if the signature is verified. Returns false otherwise.
	func verify(_ digest: Digest, signature: UnsafeRawBufferPointer, key: Key) -> Bool
	/// Encrypt this buffer using the indicated cipher, key and iv (initialization vector).
	/// Returns a newly allocated buffer which must be freed by the caller.
	func encrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
	/// Decrypt this buffer using the indicated cipher, key and iv (initialization vector).
	/// Returns a newly allocated buffer which must be freed by the caller.
	func decrypt(_ cipher: Cipher, key: UnsafeRawBufferPointer, iv: UnsafeRawBufferPointer) -> UnsafeMutableRawBufferPointer?
	/// Encrypt this buffer to PEM encoded CMS format using the indicated cipher, password, and salt.
	/// Returns a newly allocated buffer which must be freed by the caller.
	func encrypt(_ cipher: Cipher,
	             password: UnsafeRawBufferPointer,
	             salt: UnsafeRawBufferPointer,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> UnsafeMutableRawBufferPointer?
	/// Decrypt this PEM encoded CMS buffer using the indicated password and salt.
	/// Returns a newly allocated buffer which must be freed by the caller.
	func decrypt(_ cipher: Cipher,
	             password: UnsafeRawBufferPointer,
	             salt: UnsafeRawBufferPointer,
	             keyIterations: Int = 2048,
	             keyDigest: Digest = .md5) -> UnsafeMutableRawBufferPointer?
}
```

It's very important to adhere to the proper memory ownership guidelines when using these functions. Any `UnsafeMutableRawBufferPointer` returned by one of these functions must at some point be deallocated by the caller. All such return buffers will properly have their `.count` properties set to indicate their size. No `UnsafeRawBufferPointer` passed into a function will be deallocated or otherwise modified.

## JSON Web Tokens (JWT)

This crypto package provides an means for creating new JWT tokens and validating existing tokens.

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA. Source: [JWT](https://jwt.io/introduction/).

New JWT tokens are created through the `JWTCreator` object.

```swift
/// Creates and signs new JWT tokens.
public struct JWTCreator {
	/// Creates a new JWT token given a payload.
	/// The payload can then be signed to generate a JWT token string.
	public init?(payload: [String:Any])
	/// Sign and return a new JWT token string using an HMAC key.
	/// Additional headers can be optionally provided.
	/// Throws a JWT.Error.signingError if there is a problem generating the token string.
	public func sign(alg: JWT.Alg, key: String, headers: [String:Any] = [:]) throws -> String
	/// Sign and return a new JWT token string using the given key.
	/// Additional headers can be optionally provided.
	/// The key type must be compatible with the indicated `algo`.
	/// Throws a JWT.Error.signingError if there is a problem generating the token string.
	public func sign(alg: JWT.Alg, key: Key, headers: [String:Any] = [:]) throws -> String
}
```

Existing JWT tokens can be validated through the `JWTVerifier` object.

```swift
/// Accepts a JWT token string and verifies its structural validity and signature.
public struct JWTVerifier {
	/// The headers obtained from the token.
	public var header: [String:Any]
	/// The payload carried by the token.
	public var payload: [String:Any]
	/// Create a JWTVerifier given a source string in the "aaaa.bbbb.cccc" format.
	/// Returns nil if the given string is not a valid JWT.
	/// *Does not perform verification in this step.* Call `verify` with your key to validate.
	/// If verification succeeds then the `.headers` and `.payload` properties can be safely accessed.
	public init?(_ jwt: String)
	/// Verify the token based on the indicated algorithm and HMAC key.
	/// Throws a JWT.Error.verificationError if any aspect of the token is incongruent.
	/// Returns without any error if the token was able to be verified.
	/// The parameter `algo` must match the token's "alg" header.
	public func verify(algo: JWT.Alg, key: String) throws
	/// Verify the token based on the indicated algorithm and key.
	/// Throws a JWT.Error.verificationError if any aspect of the token is incongruent.
	/// Returns without any error if the token was able to be verified.
	/// The parameter `algo` must match the token's "alg" header.
	/// The key type must be compatible with the indicated `algo`.
	public func verify(algo: JWT.Alg, key: Key) throws
}
```

The following example will create and then verify a token using the "HS256" alg scheme.

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

It's important to note that the JWTVerifier will verify that the token is cryptographically sound, but it **does not** validate payload claims such as iss(uer) or exp(iration). You can obtain these from the payload dictionary and validate according to the needs of your application. 

## Algorithms

The available encoding, digest and cipher algorithms are enumerated in the `Encoding`, `Digest` and `Cipher` enums, respectively.

* Encoding: base64, base64url, hex
* Digest: md4, md5, sha, sha1, dss, dss1, ecdsa, sha224, sha256, sha384, sha512, ripemd160, whirlpool, custom(String)
* Cipher: des\_ecb, des\_ede, des\_ede3, des\_ede\_ecb, des\_ede3\_ecb, des\_cfb64, des\_cfb1, des\_cfb8, des\_ede\_cfb64, des\_ede3\_cfb1, des\_ede3\_cfb8, des\_ofb, des\_ede\_ofb, des\_ede3\_ofb, des\_cbc, des\_ede\_cbc, des\_ede3\_cbc, desx\_cbc, des\_ede3\_wrap, rc4, rc4\_40, rc4\_hmac\_md5, rc2\_ecb, rc2\_cbc, rc2\_40\_cbc, rc2\_64\_cbc, rc2\_cfb64, rc2\_ofb, bf\_ecb, bf\_cbc, bf\_cfb64, bf\_ofb, cast5\_ecb, cast5\_cbc, cast5\_cfb64, cast5\_ofb, aes\_128\_ecb, aes\_128\_cbc, aes\_128\_cfb1, aes\_128\_cfb8, aes\_128\_cfb128, aes\_128\_ofb, aes\_128\_ctr, aes\_128\_ccm, aes\_128\_gcm, aes\_128\_xts, aes\_128\_wrap, aes\_192\_ecb, aes\_192\_cbc, aes\_192\_cfb1, aes\_192\_cfb8, aes\_192\_cfb128, aes\_192\_ofb, aes\_192\_ctr, aes\_192\_ccm, aes\_192\_gcm, aes\_192\_wrap, aes\_256\_ecb, aes\_256\_cbc, aes\_256\_cfb1, aes\_256\_cfb8, aes\_256\_cfb128, aes\_256\_ofb, aes\_256\_ctr, aes\_256\_ccm, aes\_256\_gcm, aes\_256\_xts, aes\_256\_wrap, aes\_128\_cbc\_hmac\_sha1, aes\_256\_cbc\_hmac\_sha1, aes\_128\_cbc\_hmac\_sha256, aes\_256\_cbc\_hmac\_sha256, camellia\_128\_ecb, camellia\_128\_cbc, camellia\_128\_cfb1, camellia\_128\_cfb8, camellia\_128\_cfb128, camellia\_128\_ofb, camellia\_192\_ecb, camellia\_192\_cbc, camellia\_192\_cfb1, camellia\_192\_cfb8, camellia\_192\_cfb128, camellia\_192\_ofb, camellia\_256\_ecb, camellia\_256\_cbc, camellia\_256\_cfb1, camellia\_256\_cfb8, camellia\_256\_cfb128, camellia\_256\_ofb, seed\_ecb, seed\_cbc, seed\_cfb128, seed\_ofb, custom(String)

The Digest and Cipher enums provide a `custom` case which can be used to indicate the digest or encoding by name. This can be useful if your system includes additional algorithms which are not made explicitly available by this package. All of the digest and cipher algorithms are implemented by the underlaying crypto library (OpenSSL). Some of the encodings are implemented by the underlying library but others may be implemented directly by this package (hex, for example).

