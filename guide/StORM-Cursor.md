# StORM Cursor

In databse terms, a cursor defines the size and location of the returned rows within the context of the complete found set.

For a request using StORM, the cursor govens the number of rows returned and the position of the cursor in the total possible result set.

For the response, this information is echoed but the total number of possible rows in the found set is also populated.

Practically, this means that for a request you set the number of rows and the offset from the first record that you wish to retrieve:

``` swift 
let thisCursor = StORMCursor(
	limit: 50,
	offset: 100
	)
```

And in response the number of rows, the offset position and the total count is echoed back. This allows you to calculate pagination through the found set as required.

``` swift 
print(thisCursor.limit)
// 50
print(thisCursor.offset)
// 100
print(thisCursor.totalRecords)
// 1045
```

This cursor object is passed to a `.select` method, and returned as part of all objects using the database-specific StORM class inheritance. 