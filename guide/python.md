# Perfect - Python

This project provides an expressway to import Python 2.7 module as a Server Side Swift Library.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project, but can also run independently.

Ensure you have installed and activated the latest Swift 4.0 tool chain.

## Linux Build Note

Please make sure libpython2.7-dev was installed on Ubuntu 16.04:

```
$ sudo apt-get install libpython2.7-dev
```

## MacOS Build Note

Please make sure Xcode 9.0 or later version was installed.

## Quick Start

Add PerfectPython dependency to your Package.swift

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Python.git", majorVersion: 3)
```

Then import two different libraries into the swift source code:

``` swift
import PythonAPI
import PerfectPython
```

Before any python api calls, make sure to initialize the library by calling `Py_Initialize()` function:

``` swift
Py_Initialize()
```

### Import Python Modules

Use `PyObj` class to import python modules. In the following example, a python script `/tmp/clstest.py` has been imported into the current Swift context:

``` swift
let pymod = try PyObj(path: "/tmp", import: "clstest")
```

### Access Python Variables

Once imported modules, you can use `PyObj.load()` function to access a variable value, or using `PyObj.save()` to store a new value to the current python variable.

For example, if there is a variable called `stringVar` in a python script:

``` python
stringVar = 'Hello, world'
```

Then you can read its value in such a form:

``` swift
if let str = pymod.load("stringVar")?.value as? String {
	print(str)
	// will print it out as "Hello, world!"
}
```

You can also directly overwrite the value of the same variable:

``` swift
try pymod.save("stringVar", newValue: "Hola, 🇨🇳🇨🇦！")
```


**NOTE** Currently, Perfect-Python supports the following data types between Swift and Python:

Python Type|Swift Type|Remark
----------|---------|-------
int|Int|
float|Double|
str|String|
list|[Any]|Recursively
dict|[String:Any]|Recursively

For example, you can convert a Swift `String` to `PyObj` by: `let pystr = "Hello".python()` or `let pystr = try PyObj(value:"Hello")`.

To convert a `PyObj` to a Swift data type, e.g., a `String`, there are also two available approaches: `let str = pystr.value as? String` and `let str = String(python: pystr)`.


### Call A Python Function

Method `PyObj.call()` is available to execute function call with arguments. Consider the python code below:

``` python
def mymul(num1, num2):
	return num1 * num2
```

Perfect-Python can wrap this call by its name as a string and the arguments as an array:

``` swift
if let res = pymod.call("mymul", args: [2,3])?.value as? Int {
	print(res)
	// the result will be 6
}
```

### Python Object Classes

The same `PyObj.load()` function helps to access the python class type, however, a following method `PyObj.construct()` should be called for object instance initialization. This method also supports parameters as an array for python object class construction.

Assume that there is a typical python class called `Person`, which has two properties `name` and `age`, and an object method called `intro()`:

``` python
class Person:
	def __init__(self, name, age):
		self.name = name
		self.age = age
		
	def intro(self):
		return 'Name: ' + self.name + ', Age: ' + str(self.age)
```

To initialize such a class object in Swift, the first two steps look like:

``` swift
if let personClass = pymod.load("Person"),
    let person = personClass.construct(["rocky", 24]) {
    // person is now the object instance
  }
```

Then you can access the properties and class methods as common variables and functions do:

``` swift
if let name = person.load("name")?.value as? String,
    let age = person.load("age")?.value as? Int,
    let intro = person.call("intro", args: [])?.value as? String {
      print(name, age, intro)
}
```

### Callbacks

Consider the following python code as you can execute a function as a parameter like `x = caller('Hello', callback)`:

``` python
def callback(msg):
    return 'callback: ' + msg

def caller(info, func):
    return func(info)
```

The equivalent Swift code is nothing special but using the objective callback function as an argument before calling:

``` swift
if let fun = pymod.load("callback"),
   let result = pymod.call("caller", args: ["Hello", fun]),
   let v = result.value as? String {
   		print(v)
   	// it will be "callback: Hello"
}
```