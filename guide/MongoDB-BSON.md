# Working with BSON

BSON, short for Bin­ary JSON, is a bin­ary-en­coded seri­al­iz­a­tion of JSON-like doc­u­ments. Like JSON, BSON sup­ports the em­bed­ding of doc­u­ments and ar­rays with­in oth­er doc­u­ments and ar­rays. BSON also con­tains ex­ten­sions that al­low rep­res­ent­a­tion of data types that are not part of the JSON spec. For ex­ample, BSON has a Date type and a BinData type.

### Creating a BSON Object

There are several ways to create a new BSON object:

As an empty BSON object

``` swift
let bsonEmpty = BSON()
```

Specifying a bytes parameter creates a new BSON doc structure using the data provided. The bytes array contains a serialized BSON document.


``` swift
let bsonBytes = BSON(bytes: [UInt8])
```

With a JSON parameter, a new BSON document will be created from the JSON structure provided.

``` swift
let bsonJSON = BSON(json: String)
```
When specifying a document parameter containing BSON data, a new document will be created with the specified BSON data.

``` swift
let bsonBSON = BSON(document: BSON)
```

### Closing the BSON Object

To close, destroy and release the current BSON document:

``` swift 
let bson = BSON()
defer {
	bson.close()
}
```

### Converting a BSON Object to a JSON String

Use the `asString` method to convert the BSON document to an extended JSON string.

See [http://docs.mongodb.org/manual/reference/mongodb-extended-json/](http://docs.mongodb.org/manual/reference/mongodb-extended-json/) for more information on extended JSON.

``` swift
let bson.close = BSON(document: <BSON>)
defer {
	bson.close()
}
print(bson.asString())
```

Use `asArrayString` for outermost arrays:

``` swift
print(bson.asArrayString())
```

### Converting a BSON Object to a Byte Array

Use the `asBytes` method to convert the BSON document to a `[UInt8]` byte array.

``` swift
let bytesArray = bson.asBytes()
```

### Appending Data to the BSON Document

Using the `append` method, data can be added to the BSON document.

The `append` method utilizes the `@discardableResult` property. The result of the operation can be returned or ignored.

If the result of the `append` operation is returned:

* **true:** successful append
* **false:** append would overflow max size

#### Variations

Appends a new field to the BSON document of type `BSON_TYPE_OID` using the contents of oid.

``` swift
bson.append(key: <String>, oid: <bson_oid_t>)
```

Appends a new field to the BSON document of type `BSON_TYPE_DATE_TIME` using the contents of dateTime.

``` swift
bson.append(key: <String>, dateTime: <Int64>)
```

Appends a new field to the BSON document of type `BSON_TYPE_DATE_TIME` using the contents of time, a time_t @value for the number of seconds since UNIX epoch in UTC.

``` swift
bson.append(key: <String>, time: <time_t>)
```

Appends a new field to the BSON document of type `BSON_TYPE_DOUBLE` using the contents of double.

``` swift
bson.append(key: <String>, double: <Double>)
```

Appends a new field to the BSON document of type `BSON_TYPE_BOOL` using the contents of bool.

``` swift
bson.append(key: <String>, bool: <Bool>)
```

Appends a new field to the BSON document of type UTF-8 encoded string using the contents of string.

``` swift
bson.append(key: <String>, string: <String>)
```

Appends a bytes buffer to the BSON document

``` swift
bson.append(key: <String>, bytes: <[UInt8]>)
```

Appends a new field to self.doc of type `BSON_TYPE_REGEX`. @regex should be the regex string. @options should contain the options for the regex.

Valid options for @options are:
 
 *   'i' for case-insensitive
 *   'm' for multiple matching
 *   'x' for verbose mode
 *   'l' to make \w and \W locale dependent
 *   's' for dotall mode ('.' matches everything)
 *   'u' to make \w and \W match unicode

For more information on what comprimises a BSON regex, see [bsonspec.org](bsonspec.org).

 * **key:** The key of the field
 * **regex:** The regex to append to the BSON
 * **options:** Options for @regex


``` swift
bson.append(key: <String>, regex: <String>, options: <String>)
```

### Appending Arrays to a BSON Document

The `appendArray` method will append a complete array to the BSON document. BSON arrays are like documents where the key is the string version of the index. For example, the first item of the array would have the key "0". The second item would have the index "1".

``` swift
bson.appendArray(key: <String>, array: <BSON>)
```

It is also possible to begin an array append operation, then finish it after further processing, using `appendArrayBegin` and `appendArrayEnd`

`appendArrayBegin` appends a new field named key to the BSON document, the field is, however, incomplete. @child will be initialized so that you may add fields to the child array. Child will use a memory buffer owned by BSON document, and therefore, grow the parent buffer as additional space is used. This allows a single malloc'd buffer to be used when building arrays which can help reduce memory fragmentation.
     
The type of @child will be `BSON_TYPE_ARRAY`, and therefore, the keys inside of it MUST be "0", "1", etc.

``` swift
bson.appendArrayBegin(key: <String>, child: <BSON>)
```

`appendArrayEnd` finishes the appending of an array to the BSON dcoument. Child is considered disposed after this call and should not be used any further.

``` swift
bson.appendArrayEnd(key: <String>, child: <BSON>)
```

### Concatenate BSON Documents

Use `concat` to concatenate the `src` paramenter contents with the BSON document.

``` swift
bson.concat(src: <BSON>)
```

### Count Elements in BSON Document

Returns the number of elements found in the BSON document.

``` swift
bson.countKeys()
```


### Checks to See if BSON Document Contains a Field Named @key

This function is case-sensitive. Returns `true` if the key was found, otherwise `false`.

``` swift
let hasKey = bson.hasField(key: <String>)
```

### Compare Two BSON Documents for Equality

``` swift
guard let bson == bson2 else {
	return false
}
```

### Compare Two BSON Documents for Sort Priority

Returns `true` if lhs sorts above rhs, `false` otherwise

``` swift
guard let bson < bson2 else {
	return false
}
```



