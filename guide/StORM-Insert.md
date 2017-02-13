# Inserting a Row with StORM

In addition to the `.save()` method, StORM provides access to a more granular set of `.insert` functionality.

This can be used to rapidly insert larger number of rows without having to populate each property in advance.

It is also worth noting that StORM's `.save` functions are effectively convenience functions for `.insert`.

The three "forms" of `.insert` are:

``` swift
insert([(String, Any)])
insert(cols: [String], params: [Any])
insert(cols: [String], params: [Any], idcolumn: String)
```

`insert([(String, Any)])` will take the name/value pairs and insert them into the table. It will infer the id column and return its value.

`insert(cols: [String], params: [Any])` performs the same operation, but with different input supplied.

`insert(cols: [String], params: [Any], idcolumn: String)` allows you to specify the primary key column. The resulting primary key is returned.

In each case, if the primary key value is specified, a successful insert will echo the supplied value.

## Using Insert

To insert a new row and return the auto-generated primary key:

``` swift
var obj = User(connect)
obj.id = try obj.insert(
	cols: ["firstname","lastname","email"], 
	params: ["Donkey", "Kong", "donkey.kong@mailinator.com"]
	) as! Int
```

To insert a new row and return the supplied primary key:

``` swift
var obj = User(connect)
obj.id = try obj.insert(
	cols: ["id","firstname","lastname","email"], 
	params: ["10001","Donkey", "Kong", "donkey.kong@mailinator.com"]
	) as! Int
```