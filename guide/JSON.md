## JSON Converter

Perfect makes encoding and decoding JSON data very easy to implement

First, ensure the PerfectLib is imported in your Swift file:

``` swift
import PerfectLib
```

You are now able to use the JSONConvertible file to encode and decode information to and from JSON

### Encoding To JSON Data

You can convert a JSON object from the following data types:

* Array: [Any]
* Dictionary: String:Any
* String

All you need to do is call the function jsonEncodedString() for the corresponding data type you are converting.  If you are wondering, JSONConvertible adds this function to the data types listed above.  Call the function to encode JSON data like so:

``` swift
//Initialize encoded variable:  This is going to contain your encoded data in a JSON formatted String
var encoded = ""
let scoreArray : [String : Any] = ["1st Place" : 300, "2nd Place" : 230.45, "3rd Place" : 150] //Array of dictionary objects

//Set up a do-catch statement:  If the result of the try statement is successful, then the data will be encode. If not, then the code in the catch statement(print(error)) will be called

do {
encoded = try scoreArray.jsonEncodedString()
} catch let error {
print(error)		
}

print(encoded)

```

If you ran the above code, you will see the following outputted to your console in Xcode:

``` swift
{"2nd Place":230.45,"1st Place":300,"3rd Place":150}
```

### Decoding JSON Data To Usable Data Types

Try and decode the data you just encoded by using jsonDecode()

``` swift
var decoded: [String:Any]?
do {
decoded = try encoded.jsonDecode() as? [String:Any]
} catch let error {
print(error)
return
}
print(decoded)
```

After running the above code, the following is outputted to your console in Xcode:

``` swift
["2nd Place": 230.44999999999999, "1st Place": 300, "3rd Place": 150]
```

One caveat to decoding JSON objects is that you have to know the type of object being transmitted through JSON.  In the example above, we knew from the previous example that the encoded data was an Array of Dictionaries, thus the decoded variable's type was initialized as an array of dictionaries.

#### Using the Decoded Data

So now that you have your decoded data from JSON, you can iterate through the result and assign the values to variables/class properties.

``` swift
var firstPlace = Int()
var secondPlace = Double()
var thirdPlace = Int()

for result in decoded! {
switch result.key {
case "1st Place":
firstPlace = result.value as! Int
case "2nd Place":
secondPlace = result.value as! Double
case "3rd Place":
thirdPlace = result.value as! Int
default:
break
}
}

print("The top scores are: \r" + "First Place: " + "\(firstPlace)" + " Points\r" + "Second Place: " + "\(secondPlace)" + " Points\r" + "Third Place: " + "\(thirdPlace)" + " Points")
```

The console will display the following:

``` swift
The top scores are: 
First Place: 300 Points
Second Place: 230.45 Points
Third Place: 150 Points
```
#### Decoding Empty Values From JSON Data

What if the data the was returned was an empty array or dictionary?  

Lets say you retrieved the following data:

``` swift 
{"1st Place":300,"2nd Place":230.45,"3rd Place":150,"4th place":null}
```

The 4th place value is null.  Don't worry!  Perfect handles this :)

These null data values are converted into JSONConvertibleNull() objects, making it simple to handle empty data values.

For example:

``` swift
var decoded = [String:Any]()

do {
decoded = try "{\"1st Place\":300,\"4th place\":null,\"2nd Place\":230.45,\"3rd Place\":150}".jsonDecode() as! [String:Any]
print(decoded)
} catch let error {
print(error)
return
}

var firstPlace = Int()
var secondPlace = Double()
var thirdPlace = Int()

var resultArray = [String:Any]()

for result in decoded {
if !(result.value is JSONConvertibleNull) {
resultArray[result.key] = result.value
}
}

print(resultArray)
```
The console output of this code is the following:

```swift
["2nd Place": 230.44999999999999, "1st Place": 300, "3rd Place": 150]
```

By checking if the result is not a JSONConvertibleNull value, we can handle the data accordingly.  In this example, we omitted it from our printed output.

### JSON Convertible Object



### JSON Conversion Error
