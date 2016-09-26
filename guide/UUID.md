# UUID
Also known as a Globally Unique Identifier (GUID), a Universal Unique Identifier (UUID) is a 128-bit number used to uniquely identify some object or entity. The UUID relies upon a combination of components to ensure uniqueness.

### Create a New UUID object

A new UUID object can either be randomly generated, or assigned.

To randomly generate a v4 UUID:

``` swift
let u = UUID()
```

To assign a v4 UUID from a string:

``` swift
let u = UUID(<String>)
```

If the string is invalid, the object is assigned the following UUID instead: `00000000-0000-0000-0000-000000000000`

To return the string value of a UUID:

``` swift
let u1 = UUID()
print(u1.string)
```
