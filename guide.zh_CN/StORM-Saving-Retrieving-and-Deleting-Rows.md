# 数据记录的创建、保存、查询和删除

在查看本章内容时，请首先查看关于[设置数据类](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide.zh_CN/StORM/Setting-up-a-class.md)中的“User”数据类

## 创建并更新结果记录集

创建和更新数据的操作非常类似：如果主索引字段被设置为0或空，`save`方法会自动增加一个数据行（SQL INSERT操作），否则会进行数据更新（SQL UPDATE操作）。

创建一个新数据行：

``` swift
let obj 		= User(connect)
obj.firstname 	= "Joe"
obj.lastname 	= "Smith"

try obj.save {
	id in obj.id = id as! Int
}
```

注意新数据行中的属性`.email`在开始保存的时候被忽略了。此时，如果填写这个属性并在此保存：

``` swift
email = "joe.smith@example.com"
try obj.save()
```

因为 `.id` 属性已经被设置了，这时的数据保存则是一个 update 操作。

### 在主索引存在的情况下创建一个新数据记录

很多时候您希望手工设置主索引字段，在这种情况下，一旦改变了主索引字段的属性值，上面的保存工作方式就失效了。

因此，这种情况下请调用`.create()`方法。

该方法强制调用一个INSERT操作，并使用您主动设置的主索引值。

``` swift
let obj 		= User(connect)
obj.id			= 10001
obj.firstname	= "Mister"
obj.lastname	= "PotatoHead"
obj.email		= "potato@example.com"
try obj.create()
```

### 错误捕获

`.save`方法调用时可以使用`try`语法，但是 *⚠️注意⚠️* 捕获到的结果是可以被忽略的。尽管如此，有针对性的错误处理总是一个好习惯。

前文提到的`.save`可以按照如下方法进行处理：

``` swift
do {
	try obj.save {id in obj.id = id as! Int }
} catch {
	// 在控制台输出错误异常
	print("发现错误 \(error)")
	// 在此处进行错误处理。
}
```

## 查询数据

有三种方法可以用于查询一行或多行结果记录集：`.get`、`.find`和`.select`

### Get 方法

`.get`方法能够用于通过主索引值直接提取具体的数据记录。

``` swift
let obj = User(connect)
try obj.get(1)
print("用户姓名： \(obj.firstname) \(obj.lastname)")
```

用户的 id 标识也可以在 `.get` 操作之前设置。

``` swift
let obj 	= User(connect)
obj.id 		= 2
try obj.get()
print("用户姓名：\(obj.firstname) \(obj.lastname)")
```

### Find 方法

`.find` 方法会按照字段/值的方法精确匹配一个查询结果记录集。

``` swift
let obj = User(connect)
try obj.find([("firstname", "Joe")])
print("找到记录：  \(obj.id), \(obj.firstname), \(obj.lastname)")
```

### 内建的 SELECT 方法

更强大的查询功能是`.select`方法，用于获得更加复杂的查询操作。

select 方法能够接受几种不同的输入形式，最简单的一种是：

``` swift
try obj.select(
	whereclause: "firstname = $1",
	params: ["Joe"],
	orderby: ["id"]
	)
```

The `.select` 方式的好处是能够防止绑定到项目的参数被黑客进行SQL注入。尽管所有的方法都是能够达到防范目的，但是只有少数核心方法需要您了解参数绑定是如何操作的。

前面的例子中，参数`$1`用于告诉计算机该参数将会被替换为参数数组 `.params` 的第一个元素。替换参数将保护数据类型不被滥用。

而`.select`方法允许更多精准的检索，比如 LIKE 和其他逻辑比较运算符。这方面您需要事先了解以下 SQL 的基础知识。

`.select` 方法还包括一些选项：

``` swift
columns:		[String],
cursor:			StORMCursor
```

* `columns` 列数组允许为查询结果集指定字段名。
* `cursor` 是一个 `StORMCursor` 游标对象，用于确定从返回的结果记录集的首记录开始的数据行号，经常用于结果分页输出。


## 删除数据行。

删除数据的操作和`.get`方法很相似：直接确定待删除的主索引号，或者根据当前数据行号进行删除。

``` swift
let obj = User(connect)
try obj.delete(1)
```

``` swift
let obj = User(connect)
obj.id = 1
try obj.delete()
```
