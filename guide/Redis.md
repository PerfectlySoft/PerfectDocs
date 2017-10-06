# Redis

Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache, and message broker.

More info at [http://redis.io](http://redis.io)


## Quick Start

Get a redis client with defaults:

```swift
RedisClient.getClient(withIdentifier: RedisClientIdentifier()) {
	c in
	do {
		let client = try c()
		...
	} catch {
		...
	}
}
```

Ping the server:

```swift
client.ping {
	response in
	defer {
		RedisClient.releaseClient(client)
	}
	guard case .simpleString(let s) = response else {
		...
		return
	}
	XCTAssert(s == "PONG", "Unexpected response \(response)")
}
```

Set/get a value:

```swift
let (key, value) = ("mykey", "myvalue")
client.set(key: key, value: .string(value)) {
	response in
	guard case .simpleString(let s) = response else {
		...
		return
	}
	client.get(key: key) {
		response in
		defer {
			RedisClient.releaseClient(client)
		}
		guard case .bulkString = response else {
			...
			return
		}
		let s = response.toString()
		XCTAssert(s == value, "Unexpected response \(response)")
	}
}
```

Pub/sub:

```swift
RedisClient.getClient(withIdentifier: RedisClientIdentifier()) {
	c in
	do {
		let client1 = try c()
		RedisClient.getClient(withIdentifier: RedisClientIdentifier()) {
			c in
			do {
				let client2 = try c()
				client1.subscribe(channels: ["foo"]) {
					response in
					client2.publish(channel: "foo", message: .string("Hello!")) {
						response in
						client1.readPublished(timeoutSeconds: 5.0) {
							response in
							guard case .array(let array) = response else {
								...
								return
							}
							XCTAssert(array.count == 3, "Invalid array elements")
							XCTAssert(array[0].toString() == "message")
							XCTAssert(array[1].toString() == "foo")
							XCTAssert(array[2].toString() == "Hello!")
						}
					}
				}
			} catch {
				...
			}
		}
	} catch {
		...
	}
}
```

## Building

Add this project as a dependency in your Package.swift file.

```
.Package(url: "https://github.com/PerfectlySoft/Perfect-Redis.git", majorVersion: 3)
```

