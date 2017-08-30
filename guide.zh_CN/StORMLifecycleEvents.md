# StORM 全局全周期事件

<!--- To be used once more than 2-3 items exist -- pointless for just one
**Table of Contents:**
* [modifyValue](#modifyvalue)
-->

## modifyValue

```Swift
/* Signature: */ open func modifyValue(_: Any, forKey: String) -> Any
```

该事件在调用 `asData(_:)` 和 `asDataDict(_:)` 过程中会被触发，允许数据自定义编码。

举例：

```Swift
override func modifyValue(_ v: Any, forKey k: String) -> Any {
    if let d = v as? Date {
        return d.timestamptz
    }
    return v
}
```

该操作会将所有日期型变量转换为字符串变量，其格式为 PostgreSQL's `timestamp with timezone` 的带时区时间戳格式。

（ `timestamptz` 格式无关紧要，重点是返回的字符串）
