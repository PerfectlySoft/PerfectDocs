## JSON Converter

Perfect includes basic JSON encoding and decoding functionality. JSON encoding is provided through a series of extensions on many of the built-in Swift data types. Decoding is provided through an extension on the Swift String type.

It seems important to note that although Perfect provides this particular JSON encoding/decoding system, it is not required that your application uses it. Feel free to import your own favourite JSON-related functionality.

To utilize this system, first ensure that PerfectLib is imported:

``` swift
import PerfectLib
```

### Encoding To JSON Data

You can convert any of the following types directly into JSON string data:

* String
* Int
* UInt
* Double
* Bool
* Array<Any>
* Dictionary<String, Any>
* Optional
* Custom classes which inherit from JSONConvertibleObject

Note that only Optionals which contain any of the above types are directly convertible. Optionals which are nil will output as a JSON "null".

To encode any of these values, call the ```jsonEncodedString()``` function which is provided as an extension on the objects. This function may potentially throw a ```JSONConversionError.notConvertible``` error.

Example:

``` swift
let scoreArray: [String:Any] = ["1st Place": 300, "2nd Place": 230.45, "3rd Place": 150]
let encoded = try scoreArray.jsonEncodedString()
```

The result of the encoding would be the following String:

```
{"2nd Place":230.45,"1st Place":300,"3rd Place":150}
```

### Decoding JSON Data

String objects which contain JSON string data can be decoded by using the ```jsonDecode()``` function. This function can throw a ```JSONConversionError.syntaxError``` error if the String does not contain valid JSON data.

``` swift
let encoded = "{\"2nd Place\":230.45,\"1st Place\":300,\"3rd Place\":150}"
let decoded = try encoded.jsonDecode() as? [String:Any]
```

Decoding the String will produce the following dictionary:

``` swift
["2nd Place": 230.44999999999999, "1st Place": 300, "3rd Place": 150]
```

Though decoding a JSON string can produce any of the permitted values, it is most common to deal with JSON objects (dictionaries) or arrays. You will need to cast the resulting value to the expected type.

#### Using the Decoded Data

Because decoded dictionaries or arrays are always of type [String:Any] or [Any], respectively, you will need to cast the contained values to usable types. For example:

``` swift
var firstPlace = 0
var secondPlace = 0.0
var thirdPlace = 0

let encoded = "{\"2nd Place\":230.45,\"1st Place\":300,\"3rd Place\":150}"
guard let decoded = try encoded.jsonDecode() as? [String:Any] else {
	return
}

for (key, value) in decoded {
	switch key {
	case "1st Place":
		firstPlace = value as! Int
	case "2nd Place":
		secondPlace = value as! Double
	case "3rd Place":
		thirdPlace = value as! Int
	default:
		break
	}
}

print("The top scores are: \r" + "First Place: " + "\(firstPlace)" + " Points\r" + "Second Place: " + "\(secondPlace)" + " Points\r" + "Third Place: " + "\(thirdPlace)" + " Points")
```

The output would be the following:

``` swift
The top scores are:
First Place: 300 Points
Second Place: 230.45 Points
Third Place: 150 Points
```

#### Decoding Empty Values from JSON Data

As JSON null values are untyped, the system will substitute a ```JSONConvertibleNull``` in place of all JSON nulls. Example:

``` swift
let jsonString = "{\"1st Place\":300,\"4th place\":null,\"2nd Place\":230.45,\"3rd Place\":150}"

if let decoded = try jsonString.jsonDecode() as? [String:Any] {
	for (key, value) in decoded {
		if let value as? JSONConvertibleNull {
			print("The key \"\(key)\" had a null value")
		}
	}
}
```

The output would be:

```
The key "4th place" had a null value
```

### JSON Convertible Object

Perfect's JSON system provides the facilities for encoding and decoding custom classes. Any eligable class must inherit from the JSONConvertibleObject base class which is defined as follows:

```swift
/// Base for a custom object which can be converted to and from JSON.
public class JSONConvertibleObject: JSONConvertible {
    /// Default initializer.
    public init() {}
    /// Get the JSON keys/value.
    public func setJSONValues(_ values:[String:Any]) {}
    /// Set the object properties based on the JSON keys/values.
    public func getJSONValues() -> [String:Any] { return [String:Any]() }
    /// Encode the object into JSON text
    public func jsonEncodedString() throws -> String {
        return try self.getJSONValues().jsonEncodedString()
    }
}
```

Any object wishing to be JSON encoded/decoded must first register itself with the system. This registration should take place once when your application starts up. Call the ```JSONDecoding.registerJSONDecodable``` function to register your object. This function is defined as follows:

```swift
public class JSONDecoding {
	/// Function which returns a new instance of a custom object which will have its members set based on the JSON data.
	public typealias JSONConvertibleObjectCreator = () -> JSONConvertibleObject
	static public func registerJSONDecodable(name: String, creator: JSONConvertibleObjectCreator)
}
```

Registering an object requires a unique name which can be any string provided it is unique. It also requires a "creator" function which returns a new instance of the object in question.

When the system encodes a ```JSONConvertibleObject``` it calls the object's ```getJSONValues``` function. This function should return a [String:Any] dictionary containing the names and values for any properties which should be encoded into the resulting JSON string. This dictionary **must** also contain a value identifying the object type. The value must match the name by which the object was originally registered. The dictionary key for the value is identified by the ```JSONDecoding.objectIdentifierKey``` property.

When the system decodes such an object, it will find the ```JSONDecoding.objectIdentifierKey``` value and look up the object creator which had been previously registered. It will create a new instance of the type by calling that function, and will then call the new object's ```setJSONValues(_ values:[String:Any])``` function. It will pass in a dictionary containing all of the deconverted values. These values will match those previously returned by the ```getJSONValues``` function when the object was first converted. Within the ```setJSONValues``` function, the object should retreive all properties which it wants to reinstate.

The following example defines a custom ```JSONConvertibleObject ``` and converts it to a JSON string. It then decodes the object and compares it to the original. Note that this example object calls the convenience function ```getJSONValue```, which will pull a named value from the dictionary and permits providing a default value which will be returned if the dictionary does not contain the indicated key.

This example is split up into several sections.

Define the class:

```swift
class User: JSONConvertibleObject {
	static let registerName = "user"
	var firstName = ""
	var lastName = ""
	var age = 0
	override func setJSONValues(_ values: [String : Any]) {
		self.firstName = getJSONValue(named: "firstName", from: values, defaultValue: "")
		self.lastName = getJSONValue(named: "lastName", from: values, defaultValue: "")
		self.age = getJSONValue(named: "age", from: values, defaultValue: 0)
	}
	override func getJSONValues() -> [String : Any] {
		return [
			JSONDecoding.objectIdentifierKey:User.registerName,
			"firstName":firstName,
			"lastName":lastName,
			"age":age
		]
	}
}
```
Register the class:

```swift
// do this once
JSONDecoding.registerJSONDecodable(name: User.registerName, creator: { return User() })
```
Encode the object:

```swift
let user = User()
user.firstName = "Donnie"
user.lastName = "Darko"
user.age = 17

let encoded = try user.jsonEncodedString()
```
The value of "encoded" will look as follows:

```
{"lastName":"Darko","age":17,"_jsonobjid":"user","firstName":"Donnie"}
```
Decode the object:

```swift
guard let user2 = try encoded.jsonDecode() as? User else {
	return // error
}

// check the values
XCTAssert(user.firstName == user2.firstName)
XCTAssert(user.lastName == user2.lastName)
XCTAssert(user.age == user2.age)
```

### JSON Conversion Error

As an object is converted to or from a JSON string, the process may throw a ```JSONConversionError```. This is defined as follows:

```swift
/// An error occurring during JSON conversion.
public enum JSONConversionError: ErrorProtocol {
    /// The object did not suppport JSON conversion.
    case notConvertible(Any)
    /// A provided key was not a String.
    case invalidKey(Any)
    /// The JSON text contained a syntax error.
    case syntaxError
}
```
