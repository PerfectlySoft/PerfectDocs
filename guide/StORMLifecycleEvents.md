# StORM Global Lifecycle Events
<!--- To be used once more than 2-3 items exist -- pointless for just one
**Table of Contents:**
* [modifyValue](#modifyvalue)
-->

## modifyValue
```Swift
/* Signature: */ open func modifyValue(_: Any, forKey: String) -> Any
```

This event is fired during calls to `asData(_:)` and `asDataDict(_:)` to allow custom encoding of the data used.

Example:
```Swift
override func modifyValue(_ v: Any, forKey k: String) -> Any {
    if let d = v as? Date {
        return d.timestamptz
    }
    return v
}
```
This will convert all `Date`s to `String`s formatted as PostgreSQL's `timestamp with timezone` type.
(The implementation of `timestamptz` is irrelevant other than it returns a `String`)
