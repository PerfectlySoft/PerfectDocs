# Environmental Operations

## Usage

First, ensure the `PerfectLib` is imported in your Swift file:

``` swift
import PerfectLib
```
You are now able to use the `Env` class to operate the environmental variables

### Set

- Single Variable Setting:

This statement is equal to bash command "export foo=bar"

``` swift
Env.set("foo", value: "bar")
```

- Group Setting:

It is also possible to set a group of variables in a dictionary style:

``` swift
Env.set(["foo":"bar", "koo":"kar"])
// the result is identically the same as "export foo=bar && export koo=kar"
```

### Get

- Single variable query:
	
``` swift
guard let foo = Env.get("foo") else {
	// there is no such a variable
}
```

- Single variable query with a default value:

``` swift
guard let foo = Env.get("foo", defaultValue: "bar") else {
	// there is no such a variable even with a default value??
}
```

- Query all system variables:

``` swift
let all = Env.get()
// the result of all is a dictionary [String: String]
```

### Delete

- Delete an environmental variable:


``` swift
Env.del("foo")
```