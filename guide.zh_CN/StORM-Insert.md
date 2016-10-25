# 用 StORM 插入数据

除了使用 `.save()` 方法外，StORM 提供了一个更加明确的`.insert`功能用于插入数据。

该方法能够用于快速插入大量空数据行而免于逐一设置各数据行属性。

另外请注意 StORM 的 `.save` 方法内部是调用 `.insert` 方法的。

`.insert`的三种使用形式有：

``` swift
insert([(String, Any)])
insert(cols: [String], params: [Any])
insert(cols: [String], params: [Any], idcolumn: String)
```

其中， `insert([(String, Any)])` 将根据列名/取值进行逐个插入数据表，这个操作会改变主索引id字段内容并返回字段值。

`insert(cols: [String], params: [Any])` 操作是一样的，但是输入参数不一样，是两个数组。

`insert(cols: [String], params: [Any], idcolumn: String)` 允许指定主索引所在列，执行后主索引值将被返回。

上述各种情况下只要主索引被确定了，调用时都会把数索引值一同返回。

## 使用插入方法

如果希望插入新一行数据并返回一个自动创建的主索引值，请参考以下样例：

``` swift
var obj = User(connect)
obj.id = try obj.insert(
	cols: ["firstname","lastname","email"],
	params: ["Donkey", "Kong", "donkey.kong@mailinator.com"]
	) as! Int
```

插入时也可以手工填写主索引的 id 值：

``` swift
var obj = User(connect)
obj.id = try obj.insert(
	cols: ["id","firstname","lastname","email"],
	params: ["10001","Donkey", "Kong", "donkey.kong@mailinator.com"]
	) as! Int
```
