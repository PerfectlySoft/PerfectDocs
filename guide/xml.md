# XML

Perfect provides XML & HTML parsing support via the Perfect-XML module.


## Getting Started

In addition to the PerfectLib, you will need the Perfect-XML dependency in the Package.swift file:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-XML.git", majorVersion: 3)
```

In each Swift source file that references the XML classes, include the import directive:

``` swift
import PerfectXML
```


### macOS Build Notes

If you receive a compile error that says the following, you need to install and link libxml2.

```
note: you may be able to install libxml-2.0 using your system-packager:

    brew install libxml2

Compile Swift Module 'PerfectXML' (2 sources)
<module-includes>:1:9: note: in file included from <module-includes>:1:
#import "libxml2.h"
```

To install and link libxml2 with Homebrew, use the following two commands:

```
brew install libxml2
brew link --force libxml2
```

### Linux Setup note

Ensure that you have installed libxml2-dev and pkg-config.

```
sudo apt-get install libxml2-dev pkg-config
```


## Parsing an XML string

Instantiate an XDocument object with your XML string:

``` swift
let xDoc = XDocument(fromSource: <XMLSource>)
```

Now you can get the root node of the XML structure by using the documentElement property.


## Convert the node tree to String

Converting the XML node tree to a string, with optional pretty-print formatting:

``` swift
let xDoc = XDocument(fromSource: rssXML)
let prettyString = xDoc?.string(pretty: true)
let plainString = xDoc?.string(pretty: false)
```

## Working with Nodes

XML is a structured document standard consisting of nodes in the format `<A>B</A>`. Each node has a tag name and either a value, or sub nodes we call "children". Each node has several important properties:

- nodeValue
- nodeName
- parentNode
- childNodes

Each node also has a `getElementsByTagName` method that recursively searches through it and its children to return an array of all nodes that have that name. This method makes it easy to find a single value in the XML file.

To demonstrate, this recursive function iterates through all elements in the node:

``` swift
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

printChildrenName( xDoc!)
```

### Access Node by Tag Name

The snippet below shows the method of `getElementsByTagName` in a general manner:

``` swift
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

### Access Node by ID

Alternatively querying a node by its id using `.getElementById()`:

``` swift
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

#### XElement

The method `.getElementsByTagName()` returns an array of nodes of type XElement.

The following code demonstrates how to iterate all element in this array:

``` swift
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

### Relationships of a Node Family

PerfectXML provides a convenient way to access all relationships of a specific XML node: Parent, Siblings & Children.

#### Parent of a Node

Parent node can be accessed using ```parentNode```:

``` swift
func showParent(tag: String) {

		guard let node = xDoc?.documentElement?.getElementsByTagName(tag).first else {
			print("There is no such a tag '\(tag)'.\n")
			return
		}

		// Accessing the parent node
		guard let parent = node.parentNode else {
			print("Tag '\(tag)' is the root node.\n")
			return
		}
		let name = parent.nodeName
		print("Parent of '\(tag)' is '\(name)'\n")
}

showParent(tag: "link")
```

#### Node Siblings

Each XML node can have two siblings: `previousSibling` and `nextSibling`.

``` swift
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

#### First & Last Child

If an XML node has a child, `.firstChild` and `.lastChild` can be used:

``` swift
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


## Node Attributes

Any XML node or element can have attributes:

```
<node attribute1="value of attribute1" attribute2="value of attribute2">
</node>
```

Node method `.getAttribute(name: String)` provides the functionality of accessing attributes:

``` swift
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

Both `.getElementsByTagName()` and `.getAttributeNode()` have namespace versions: `.getElementsByTagNameNS()` and `.getAttributeNodeNS()`. In these cases `namespaceURI` and `localName` are required.

The following code demonstrates the usage of `.getElementsByTagNameNS()` and `.getNamedItemNS()`:

``` swift
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

XPath (XML Path Language) is a query language for selecting nodes from an XML document. In addition, XPath may be used to compute values (e.g. strings, numbers, or Boolean values) from the content of an XML document.

The following code demonstrates extracting a specific path:

``` swift
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

 