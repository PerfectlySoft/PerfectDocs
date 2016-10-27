# StORM 记录游标

在数据库理论定义中，游标代表在当前数据环境，也就是数据库交互时返回的结果记录集（一个数据表）中，当前操作对应的行列位置。

使用 StORM 时，游标会返回结果记录集的总行数，并指向期望的行数位置。

执行游标设定操作后，StORM会响应总记录行数，但是返回的是实际查询的结果记录集行数。

实际应用中，这意味着，您可以设置希望返回的结果数量，并将游标设置为从实际返回的结果记录行作为开始的行号：

``` swift
let thisCursor = StORMCursor(
	limit: 50,
	offset: 100
	)
```

作为数据行的响应，行号（即上面的offset偏移量）和总记录数会被刷新和回响。通过这种方法您可以实现数据结果查询分页。

``` swift
print(thisCursor.limit)
// 50
print(thisCursor.offset)
// 100
print(thisCursor.totalRecords)
// 1045
```

该游标对象会被传递给`.select`查询方法，并将StORM 实现的数据类对象以结果记录集方式返回。
