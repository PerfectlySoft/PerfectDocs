# XML support for Perfect
Perfect XML Example Project

This repository demonstrates how to manipulate XML api. It currently contains most of the DOM Core level 2 *read-only* APIs and includes XPath support.

## Setup

Before start this demo, please make sure Swift 3.0 or later version has already installed properly on your system. Please check [Swift 3.0 Installation Guide] (http://swift.org) for detail.

### macOS Build Notes

If you receive a compile error that says the following, you need to install and link libxml2

```
note: you may be able to install libxml-2.0 using your system-packager:

    brew install libxml2

Compile Swift Module 'PerfectXML' (2 sources)
<module-includes>:1:9: note: in file included from <module-includes>:1:
#import "libxml2.h"
```

To install and link libxml2 with homebrew, use the following two commands

```
brew install libxml2
brew link --force libxml2
```

### Linux Setup note

Ensure that you have installed libxml2-dev and pkg-config.

```
sudo apt-get install libxml2-dev pkg-config
```

## Quick Start

The general idea of this example is to show data extracted from an XML string. In another word, if you treat XML as a database, then PerfectXML will be the database connector and XPath will be the query language. You can download this example for a quick start:

```bash
git clone https://github.com/PerfectlySoft/PerfectExample-XML.git
```

or if you can start it step by step:

```bash
mkdir PerfectExample-XML
cd PerfectExample-XML
swift package init
```

In this case, please modify the Package.swift and manually adding Perfect-libxml2 / Perfect-XML libraries like this:

```Swift
import PackageDescription

let package = Package(
	name: "PerfectExample-XML",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-libxml2.git", majorVersion: 2, minor: 0),
		.Package(url: "https://github.com/PerfectlySoft/Perfect-XML.git", majorVersion: 2, minor: 0)
    ]
)
```

Please also include PerfectXML library in your swift code:

```Swift
import PerfectXML
```

Then you can use Swift Package Manager to build it up & run:

```bash
swift build
./.build/debug/PerfectExample-XML
```

## A Sample XML content

Before doing any actual accessing operation, please copy the following XML sample string to your code, or you can make a similar file stored on the local directory and read it out into the same string:

```Swift
let rssXML = "<?xml version=\"1.0\" encoding=\"UTF-8\" ?>" +
"<rss version=\"2.0\">" +
"<channel>" +
"<title attribute1='first attribute' attribute2='second attribute'>W3Schools Home Page</title>" +
"<link>http://www.w3schools.com</link>" +
"<description>Free web building tutorials</description>" +
"<item id='rssID'>" +
"<title>RSS Tutorial</title>" +
"<link>http://www.w3schools.com/xml/xml_rss.asp</link>" +
"<description>New RSS tutorial on W3Schools</description>" +
"</item>" +
"<item id='xmlID'>" +
"<title>XML Tutorial</title>" +
"<link>http://www.w3schools.com/xml</link>" +
"<description>New XML tutorial on W3Schools</description>" +
"<deeper xmlns:foo='foo:bar'>" +
"<deepest foo:atr1='the value' foo:atr2='the other value'></deepest>" +
"<foo:fool><foo:silly>boo</foo:silly><foo:dummy>woo</foo:dummy></foo:fool>" +
"</deeper>" +
"</item>" +
"</channel>" +
"</rss>"
```


## Parse an XML string

Instantiate an XDocument object with your XML string

```swift
import PerfectXML
let xDoc = XDocument(fromSource: rssXML)
```

Now you can get the root node of the XML structure by using the documentElement property


## Make it pretty

Yes, I know your first impression about this above XML - ugly, right? But the good news is that now we can turn it into a pretty format:

```Swift
let xDoc = XDocument(fromSource: rssXML)
let prettyString = xDoc?.string(pretty: true)
let plainString = xDoc?.string(pretty: false)
```

The first line ``` xDoc = XDocument ``` created an XML object, and the method ```string(pretty: Bool)``` can get the text content back with an option of pretty format or not.

You must want to know how pretty it is, right? OK, paste the following snippet and check it out:

```Swift
print("Parse an XML document\n")

func showParse() {
		print("XML Parse Function\n")
		print("Orginal XML:\n\(rssXML)\n\n")
		print("Pretty:\n\(prettyString!)\n\n")
		print("Plain\n\(plainString!)\n\n")
}

showParse()

```

If success, you can run the app and then should see three different paragraphs of the same XML but in different formats. The first one is the original XML, the third one is a parsed string and the middle one is the formatted string after parsing.  

## Working with Nodes

XML is a structured documentation standard with a basic format of ```<A>B</A>``` - an XML node. Each node has a tag name, a value, or sub nodes we call "children".  Each node has several important properties

- nodeValue
- nodeName
- parentNode
- childNodes

Each node also has a getElementsByTagName: method that recursively searches through it and its children to return an array of all nodes that have that name. This method makes it easy to find a single value in the XML file.

To better understand these definitions, trying the following code to "walk through" the whole XML is highly recommended.

Firstly, we will need a recursive function to iterate all elements inside:

```Swift
func printChildrenName(_ xNode: XNode) {

	// try treating the current node as a text node
	guard let text = xNode as? XText else {

			// print out the node info
			print("Name:\t\(xNode.nodeName)\tType:\(xNode.nodeType)\t(children)\n")

			// find out children of the node
			for n in xNode.childNodes {

				// call the function recursively and take the same produces
				printChildrenName(n)
			}
			return
	}

	// it is a text node and print it out
	print("Name:\t\(xNode.nodeName)\tType:\(xNode.nodeType)\tValue:\t\(text.nodeValue!)\n")
}
```


Now we can show the whole structure of this XML file:

```Swift
printChildrenName( xDoc!)
```

## Access Node by Tag Name

A very basic way of querying a specific node is by the tag name. The snippet below shows the method of ```getElementsByTagName ``` in a general manner:

```Swift
func testTag(tagName: String) {

	// use .getElementsByTagName to get this node.
	// check if there is such a tag in the xml document
		guard let node = xDoc?.documentElement?.getElementsByTagName(tagName) else {
			print("There is no such a tag: `\(tagName)`\n")
			return
		}

		// if is, get the first matched
		guard let value = node.first?.nodeValue else {
			print("Tag `\(tagName)` has no value\n")
			return
		}

		// show the node value
		print("Tag '\(tagName)' has a value of '\(value)'\n")
}

testTag(tagName: "link")
testTag(tagName: "description")
```

## Access Node by ID

Alternatively, you may also notice that any node can have an attribute called "id", which means you can access this node without knowing its tag:

```XML
<item id='xmlID'>
```

To get this node by ID, PerfectXML provides a method called .getElementById(), so we can build another route to do the similar API as the previous example but this time we will try the id method:

```Swift
func testID(id: String) {

		// Access node by its id, if available
		guard let node = xDoc?.getElementById(id) else {
			print("There is no such a id: `\(id)`\n")
			return
		}

		guard let value = node.nodeValue else {
			print("id `\(id)` has no value\n")
			return
		}

		print("id '\(id)' has a value of '\(value)'\n")
}

testID(id: "rssID")
testID(id: "xmlID")
```

Anyway can you see the difference of .getElementById() and .getElementsByTagName() ? You can have a try to see if there are duplicated IDs in the same XML.

## More about getElementsByTagName()

Method .getElementsByTagName returns an array of nodes, i.e., [XElement], just like a record set in a database query.

The following code demonstrates how to iterate all element in this array:

```Swift
print("Show Items\n")

func showItems() {
		// get all items with tag name of "item"
		let feedItems = xDoc?.documentElement?.getElementsByTagName("item")

		// checkout how many items indeed.
		let itemsCount = feedItems?.count
		print("There are totally \(itemsCount!) Items Found\n")

		// iterate all items in the result set
		for item in feedItems!
		{
				let title = item.getElementsByTagName("title").first?.nodeValue
				let link = item.getElementsByTagName("link").first?.nodeValue
				let description = item.getElementsByTagName("description").first?.nodeValue
				print("Title: \(title!)\tLink: \(link!)\tDescription: \(description!)\n")
		}
}

showItems()
```

## Relationships of a Node Family

PerfectXML provides a convenient way to access all relationships of a specific XML node: Parent, Siblings & Children.

### Parent of a Node

Parent node can be accessed by a node property called "parentNode", see the following example to see the usage:

```Swift
print("Parent of a Node\n")

func showParent(tag: String) {

		guard let node = xDoc?.documentElement?.getElementsByTagName(tag).first else {
			print("There is no such a tag '\(tag)'.\n")
			return
		}

		// HERE is the way of accessing a parent node
		guard let parent = node.parentNode else {
			print("Tag '\(tag)' is the root node.\n")
			return
		}
		let name = parent.nodeName
		print("Parent of '\(tag)' is '\(name)'\n")
}

showParent(tag: "link")
```

### Node Siblings

Each XML node can have two siblings: previousSibling and nextSibling. Try the following snippet to test the availability of siblings.

```Swift
print("Siblings of a Node\n")

func showSiblings (tag: String) {

		let node = xDoc?.documentElement?.getElementsByTagName(tag).first

		// Check the previous sibling of current node
		let previousNode = node?.previousSibling
		var name = previousNode?.nodeName
		var value = previousNode?.nodeValue
		print("Previous Sibling of \(tag) is \(name!)\t\(value!)\n")

		// Check the next sibling of current node
		let nextNode = node?.nextSibling
		name = nextNode?.nodeName
		value = nextNode?.nodeValue
		print("Next Sibling of \(tag) is \(name!)\t\(value!)\n")
}


showSiblings(tag: "link")
showSiblings(tag: "description")
```

### First & Last Child

If an XML node has a child, then you can try properties of .firstChild / .lastChild instead of accessing them from the .childNodes array:

```Swift
print("First & Last Child\n")

func firstLast() {
		let node = xDoc?.documentElement?.getElementsByTagName("channel").first

		/// retrieve the first child
		let firstChild = node?.firstChild
		var name = firstChild?.nodeName
		var value = firstChild?.nodeValue
		print("First Child of Channel is \(name!)\t\(value!)\n")

		/// retrieve the last child
		let lastChild = node?.lastChild
		name = lastChild?.nodeName
		value = lastChild?.nodeValue
		print("Last Child of Channel is \(name!)\t\(value!)\n")
}

firstLast()
```

These methods are convenient in certain scenarios, such as jump to the first page or the last page in a book.

## Node Attributes

Any XML node / element can have attributes in such a style:

 ```XML
 <node attribute1='value of attribute1' attribute2='value of attribute2'>
   ...
 </node>
```

Node method .getAttribute(name: String) provides the functionality of accessing attributes. See the code below:

```Swift
print("Attributes of an element\n")

func showAttributes() {
		let node = xDoc?.documentElement?.getElementsByTagName("title").first

		/// get some attributes of a node
		let att1 = node?.getAttribute(name: "attribute1")
		print("attribute1 of title is \(att1)\n")

		let att2 = node?.getAttributeNode(name: "attribute2")
		print("attribute2 of title is \(att2?.value)\n")
}

showAttributes()
```

## Namespaces

XML namespaces are used for providing uniquely named elements and attributes in an XML document. An XML instance may contain element or attribute names from more than one XML vocabulary. If each vocabulary is given a namespace, the ambiguity between identically named elements or attributes can be resolved.

Both .getElementsByTagName() and .getAttributeNode() have namespace versions, i.e., .getElementsByTagNameNS() / .getAttributeNodeNS, etc. In these cases, namespaceURI and localName shall present to complete the request, as the last name and first name for a person.

The following code demonstrates the usage of .getElementsByTagNameNS() and .getNamedItemNS():

```Swift
print("Namespaces\n")

func showNamespaces(){
	let deeper = xDoc?.documentElement?.getElementsByTagName("deeper").first
	let atts = deeper?.firstChild?.attributes;
	let item = atts?.getNamedItemNS(namespaceURI: "foo:bar", localName: "atr2")
	print("Namespace of deeper has an attribute of \(item?.nodeValue)\n")

	let foos = xDoc?.documentElement?.getElementsByTagNameNS(namespaceURI: "foo:bar", localName: "fool")
	var count = foos?.count
	let node = foos?.first
	let name = node?.nodeName
	let localName = node?.localName
	let prefix = node?.prefix
	let nuri = node?.namespaceURI

	print("Namespace of 'foo:bar' has \(count!) element:\n")
	print("Node Name: \(name!)\n")
	print("Local Name: \(localName!)\n")
	print("Prefix: \(prefix!)\n")
	print("Namespace URI: \(nuri!)\n")

	let children = node?.childNodes

	count = children?.count

	let a = node?.firstChild
	let b = node?.lastChild

	let na = a?.nodeName
	let nb = b?.nodeName
	let va = a?.nodeValue
	let vb = b?.nodeValue

	print("This node also has \(count!) children.\n")
	print("The first one is called '\(na!)' with value of '\(va!)'.\n")
	print("And the last one is called '\(nb!)' with value of '\(vb!)'\n")
}

showNamespaces()

```

## XPath

XPath (XML Path Language) is a query language for selecting nodes from an XML document. In addition, XPath may be used to compute values (e.g., strings, numbers, or Boolean values) from the content of an XML document.

This demo can extract the path you input:

```Swift

print("XML Path Demo\n")

func showXPath(xpath: String) {
    /// Use .extract() method to deal with the XPath request.
		let pathResource = xDoc?.extract(path: xpath)
		print("XPath '\(xpath)':\n\(pathResource!)\n")
}

showXPath(xpath: "/rss/channel/item")
showXPath(xpath: "/rss/channel/title/@attribute1")
showXPath(xpath: "/rss/channel/link/text()")
showXPath(xpath: "/rss/channel/item[2]/deeper/foo:bar")
```

Then build & run and try any possible XPath as need. Now you can see how powerful XML & XPath are - theoretically, it can even wrap up a total file system, can't it?
