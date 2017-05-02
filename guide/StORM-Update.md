# Updating Rows with StORM

In a way similar to StORM's `.insert` methods, you can use `.update` methods to specify updates directly.

The two forms of `.update` available are:

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

The only difference between these two is the format of the data supplied.

## Using Update

Let's look at an example of how to create, then update a row:

``` swift
let obj 		= User()
obj.firstname 	= "Joe"
obj.lastname 	= "Smith"

// First, lets create a new row
try obj.save {
	id in obj.id = id as! Int
}

// Now, we change the values
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

While the above illustrates the direct update process, in this case calling `try obj.save()` would be leaner and just as efective as the `.update`.

Where an `.update` would be more effective, is a situation where you know all the information, including the primary key value, before any fetch from the datasource.

This way, an update would be called directly without any initial round trip connection to the database.

``` swift
let obj 		= User()
try obj.update(
	cols: ["firstname","lastname","email"], 
	params: ["Mickey", "Mouse", "Mickey.Mouse@mailinator.com"], 
	idName: "id",
	idValue: 100001
	)

```