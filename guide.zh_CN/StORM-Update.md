# 使用 StORM 更新数据记录

与 StORM 的`.insert`方法一样，可以使用 `.update` 方法对数据对象对应的数据记录进行更新。

具体说来，`.update`有两种形式：

``` swift
update(
	cols: [String],
	params: [Any],
	idName: String,
	idValue: Any
	)

update(
	data: [(String, Any)],
	idName: String = "id",
	idValue: Any
	)
```

二者差别只是所用变量形式不同。

## 使用 Update 方法

请参考以下示例更新数据行：

``` swift
let obj 		= User(connect)
obj.firstname 	= "Joe"
obj.lastname 	= "Smith"

// 首先，创建一行新数据
try obj.save {
	id in obj.id = id as! Int
}

// 然后再改变数据内容
obj.firstname = "Mickey"
obj.lastname = "Mouse"
obj.email = "Mickey.Mouse@mailinator.com"

try obj.update(
	cols: ["firstname","lastname","email"],
	params: [obj.firstname, obj.lastname, obj.email],
	idName: "id",
	idValue: obj.id
	)
```

上述方法介绍了如何直接更新数据的过程，这种情况下调用`try obj.save()`和 `.update`是同等效率的。

而`.update`更有效的使用环境是，在从数据源取得数据记录之前，您就实现已经掌握了主索引记录的值。

这种情况下，UPDATE操作将不经任何包装处理，直接连接到数据库完成操作。

``` swift
let obj 		= User(connect)
try obj.update(
	cols: ["firstname","lastname","email"],
	params: ["Mickey", "Mouse", "Mickey.Mouse@mailinator.com"],
	idName: "id",
	idValue: 100001
	)

```
