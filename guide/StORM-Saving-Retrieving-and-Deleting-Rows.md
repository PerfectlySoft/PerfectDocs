# Creating, Saving, Finding & Deleting

For the purposes of this documentation the class "User" from "[Setting up a class](https://github.com/PerfectlySoft/PerfectDocs/blob/master/guide/StORM-Setting-up-a-class.md)" will be used for most examples.

## Creating and Updating Rows

The act of creating and saving a record is remarkably similar: The `save` method will create a new row if the primary key property is 0 or empty, otherwise it will attempt to execute an update.

To create a new row:

``` swift
let obj 		= User()
obj.firstname 	= "Joe"
obj.lastname 	= "Smith"

try obj.save {
	id in obj.id = id as! Int
}
```

Note that the `.email` property was omitted in the initial save. Let's fix that:

``` swift
obj.email = "joe.smith@example.com"
try obj.save()
```

Because the `.id` property was set, an update is performed.

### Creating a new record that already has the Primary Key set

There are some situations where the you will wish to set the primary key, and the `.save` method above will not work in this case. 

Here, you will need to invoke the `.create()` method instead.

This forces an insert instead, using your value for the primary key.

``` swift
let obj 		= User()
obj.id			= 10001
obj.firstname	= "Mister"
obj.lastname	= "PotatoHead"
obj.email		= "potato@example.com"
try obj.create()
```

### Capturing Errors

The simple form of the `.save` uses `try`, but the result *can* be ignored. However it's good practice to trap for these errors and act accordingly. 

The initial `.save` could be handled instead like this:

``` swift
do {
	try obj.save {id in obj.id = id as! Int }
} catch {
	// Inform the developer using console logging:
	print("There was an error: \(error)")
	// Do something about it in code.
}
```

## Retrieving Rows

There are three ways to retrieve one or more rows: `.get`, `.find`, and `.select`.

### Get Methods

The `.get` methods will attempt to retrieve rows via an exact primary key column match.

``` swift
let obj = User()
try obj.get(1)
print("User's name: \(obj.firstname) \(obj.lastname)")
```

The user id can also be set before the `.get` is performed:

``` swift
let obj 	= User()
obj.id 		= 2
try obj.get()
print("User's name: \(obj.firstname) \(obj.lastname)")
```

### The Find Method

`.find` will perform an exact match on the name/value pairs supplied.

``` swift
let obj = User()
try obj.find([("firstname", "Joe")])
print("Find Record:  \(obj.id), \(obj.firstname), \(obj.lastname)")
```

### Using Select

By far the most powerful of the row retrieval methods is `.select`.

The select method will take several input forms; the simplest is:

``` swift
try obj.select(
	whereclause: "firstname = $1", 
	params: ["Joe"], 
	orderby: ["id"]
	)
```

The `.select` method takes advantage of parameter binding to protect from SQL injection. Actually all methods do, but only a few core methods expose this and require knowledge of how parameter binding works.

In the previous example, the parameter `$1` is used to signal that the first item in the array fed to `.params` should be used. The substitution will protect for data type and potential nasty input. 

The `.select` method also will allow more specific search options such as LIKE and other comparison operators. Knowledge of SQL becomes necessary beyond the basics.

The additional options available in the `.select` method are:

``` swift
columns:		[String],
cursor:			StORMCursor
```

* `columns` allows an array of column names to be supplied which will target specific data that will be returned from the query.
* `cursor` is a `StORMCursor` object that specifies the number of rows to return and the offset from the 1st record. This is often used for pagination.


## Deleting Rows

Deleting a row is done in a similar way to the `.get` method: it can be done by supplying a primary key, or by deleting the currently held row.

``` swift
let obj = User()
try obj.delete(1)
```

``` swift
let obj = User()
obj.id = 1
try obj.delete()
```
