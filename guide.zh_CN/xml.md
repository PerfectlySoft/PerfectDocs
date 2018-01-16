# Perfect XML 函数库

Perfect 通过Perfect-XML模块提供 XML 和 HTML 解析功能。


## 准备工作

除了安装 PerfectLib 函数库之外，请在您的 Package.swift 文件中设置 Perfect-XML 函数库依存关系。

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-XML.git", majorVersion: 3)
```

在需要使用 XML 解析功能的源代码开头，请用下列程序声明使用：

``` swift
import PerfectXML
```


### macOS 安装注意事项

如果您看到下列消息，可能需要使用 homebrew 安装并链接 libxml2

```
note: you may be able to install libxml-2.0 using your system-packager:

    brew install libxml2

Compile Swift Module 'PerfectXML' (2 sources)
<module-includes>:1:9: note: in file included from <module-includes>:1:
#import "libxml2.h"
```

具体 homebrew 安装 libxml2 的方法：

```
brew install libxml2
brew link --force libxml2
```

### Linux 安装注意事项

请确保您的系统已经同时安装了 libxml2-dev 和 pkg-config.

```
sudo apt-get install libxml2-dev pkg-config
```


## 解析 XML 字符串

请用 XDocument 对象来构造并解析 XML 字符串

``` swift
let xDoc = XDocument(fromSource: rssXML)
```

现在您就可以将上述变量作为 XML 结构的根节点并使用 documentElement 属性进行访问了。


## 将树状节点转化为字符串

下列方法可以把XML节点树转化为字符串，并带有格式化效果选项：

``` Swift
let xDoc = XDocument(fromSource: rssXML)
let prettyString = xDoc?.string(pretty: true)
let plainString = xDoc?.string(pretty: false)
```

## XML 节点

XML是一个结构化文本，每个形如```<A>B</A>```的结构都是一个XML节点。每个节点都包含一个标签名(tag name)、内容值或子节点。每个 XML 节点都包含下列重要属性：

- nodeValue 节点内容值
- nodeName  节点名称
- parentNode  父节点
- childNodes  子节点

同时，每个节点都有一个很重要的方法，即 getElementsByTagName —— 通过标签名称访问元素。该方法能够递归搜索整个 XML 文档并返回所有符合该名称的节点，这个方法可以用于方便地在 XML 文件内检索任何一个节点。

以下递归函数可以用来展示如何进行节点便利

``` Swift
func printChildrenName(_ xNode: XNode) {

	// 检查一下节点类型是不是文字节点
	guard let text = xNode as? XText else {

		// 如果不是就输出节点类型
		print("节点名称：\(xNode.nodeName)\t节点类型：\(xNode.nodeType)\t（包含子节点）\n")

		// 遍历每个子节点
		for n in xNode.childNodes {

			// 再次调用递归函数
			printChildrenName(n)
		}
	
		return
	}

	// 如果是文字节点就直接打印出来
	print("节点名称：\(xNode.nodeName)\t节点类型\(xNode.nodeType)\t内容值\t\(text.nodeValue!)\n")
}

printChildrenName( xDoc!)
```

### 通过标签名称访问节点

访问节点的最基本方法就是通过节点名称实现。以下程序展示了```getElementsByTagName ```的通用方法：

``` Swift
func testTag(tagName: String) {

	// 调用 .getElementsByTagName 访问节点
	// 并且检查该节点是否有效（在XML文件中存在）
	guard let node = xDoc?.documentElement?.getElementsByTagName(tagName) else {
		print("未找到标签“\(tagName)”\n")
		return
	}

	// 如果找到了，就提取首个节点作为代表
	guard let value = node.first?.nodeValue else {
		print("标签“\(tagName)”不包含内容。\n")
		return
	}

	// 显示节点内容
	print("节点“\(tagName)”内容为“\(value)”\n")
}

testTag(tagName: "link")
testTag(tagName: "description")
```

### 通过ID唯一属性标识符访问节点

另外一种便捷的方法是通过唯一标识符`.getElementById()`方法进行节点提取：

``` Swift
func testID(id: String) {

	// 如果存在适合的ID，就可以用下面的方法进行访问
	guard let node = xDoc?.getElementById(id) else {
		print("文档中不存在这样的ID“\(id)”\n")
		return
	}

	guard let value = node.nodeValue else {
		print("id名为“\(id)”的节点没有内容\n")
		return
	}

	print("节点“\(id)”内容为“\(value)”\n")
}

testID(id: "rssID")
testID(id: "xmlID")
```

#### XElement

`.getElementsByTagName`返回的是一个节点数组 [XElement]，如同数据库的一个结果记录集一样。

以下代码展示了如何将所有标签名相同的同级节点返回为一个数组：

``` Swift
func showItems() {
	// 选择所有名为“item”的节点
	let feedItems = xDoc?.documentElement?.getElementsByTagName("item")

	// 检查该数组实际元素数量
	let itemsCount = feedItems?.count
	print("总共找到 \(itemsCount!) 个元素\n")

	// 遍历结果集内所有元素
	for item in feedItems!
	{
		let title = item.getElementsByTagName("title").first?.nodeValue
		let link = item.getElementsByTagName("link").first?.nodeValue
		let description = item.getElementsByTagName("description").first?.nodeValue
		print("标题：\(title!)\t链接：\(link!)\t说明： \(description!)\n")
	}
}

showItems()
```

### 通过节点之间的关系进行访问

PerfectXML 函数库提供一系列简便方法，用于根据当前 XML 节点信息访问所有相关节点：父节点、子节点和同级相邻节点：

#### 父节点

父节点的访问可以通过当前节点的“parentNode”属性进行访问，用法如下：

``` Swift
func showParent(tag: String) {

	guard let node = xDoc?.documentElement?.getElementsByTagName(tag).first else {
		print("未找到标签名为“\(tag)”的节点。\n")
		return
	}

	// 访问父节点；如果父节点为空则意味着是根节点。
	guard let parent = node.parentNode else {
		print("标签“\(tag)”为文档根节点。\n")
		return
	}
	let name = parent.nodeName
	print("节点“\(tag)”的父节点（上一级节点）是“\(name)”。\n")
}

showParent(tag: "link")
```

#### 同级相邻节点

每个 XML 节点都可能存在两个同级相邻节点：previousSibling（前一个节点）和 nextSibling（后一个节点）。以下代码演示了相邻节点的互相访问：

``` Swift
func showSiblings (tag: String) {

	let node = xDoc?.documentElement?.getElementsByTagName(tag).first

	// 查看当前节点的前一个同级相邻节点。
	let previousNode = node?.previousSibling
	var name = previousNode?.nodeName
	var value = previousNode?.nodeValue
	print("标签“\(tag)”的前一个同级相邻节点名称为\(name!)，\t内容值为：\(value!)\n")

	// 查看当前节点的后一个同级相邻节点。
	let nextNode = node?.nextSibling
	name = nextNode?.nodeName
	value = nextNode?.nodeValue
	print("标签“\(tag)”的后一个同级相邻节点名称为\(name!)，\t内容值为：\(value!)\n")
}

showSiblings(tag: "link")
showSiblings(tag: "description")
```

#### 子节点：首个子节点和末尾子节点

如果一个 XML 节点存在子节点，那么可以尝试用 .firstChild （首个子节点）/ .lastChild （末尾子节点）属属性直接访问，而避免通过.childNodes 子节点数组去计算和访问：

``` Swift
func firstLast() {
	let node = xDoc?.documentElement?.getElementsByTagName("channel").first

	/// 返回首个子节点
	let firstChild = node?.firstChild
	var name = firstChild?.nodeName
	var value = firstChild?.nodeValue
	print("Channel的首个子节点是：\(name!)\t\(value!)\n")

	/// 返回末尾子节点
	let lastChild = node?.lastChild
	name = lastChild?.nodeName
	value = lastChild?.nodeValue
	print("Channel的末尾子节点是：\(name!)\t\(value!)\n")
}

firstLast()
```


## 节点属性

任意 XML 节点/元素都能自定义多个属性，格式如下：

```
 <node attribute1='value of attribute1' attribute2='value of attribute2'>
 </node>
```

节点对象方法 .getAttribute(name: String) 用于访问这些属性。参考以下例子：

``` swift
func showAttributes() {
	let node = xDoc?.documentElement?.getElementsByTagName("title").first

	/// 读取一个节点的若干属性
	let att1 = node?.getAttribute(name: "attribute1")
	print("标题对象的属性1“attribute1”内容为 \(att1)\n")

	let att2 = node?.getAttributeNode(name: "attribute2")
	print("标题对象的属性2“attribute2”内容为 \(att2)\n")
}

showAttributes()
```

## 命名空间

XML 规定了允许在同一个 XML文档内保证名称属性即使重复也能实现唯一性访问的方法，即命名空间。一个 XML 实例可以包含不限于同一 XML 字典约束的任意元素或属性名称。如果每个字典都有一个独立的命名空间，则区分重名元素和重名属性变得非常简单，不会导致内容混淆。

访问节点及属性的方法 .getElementsByTagName() 和 .getAttributeNode() 都有命名空间的版本，即 .getElementsByTagNameNS() 和 .getAttributeNodeNS。这种情况下，同时需要输入命名空间名称 namespaceURI 和在该空间内的本地名称 localName 以完成查询 —— 就像姓和名一样确保提取过程的唯一性。

以下代码演示了 .getElementsByTagNameNS() 和 .getNamedItemNS()的使用方法：

``` Swift
func showNamespaces(){
	let deeper = xDoc?.documentElement?.getElementsByTagName("deeper").first
	let atts = deeper?.firstChild?.attributes;
	let item = atts?.getNamedItemNS(namespaceURI: "foo:bar", localName: "atr2")
	print("deeper的命名空间内有一个属性值为“\(item?.nodeValue)”。\n")

	let foos = xDoc?.documentElement?.getElementsByTagNameNS(namespaceURI: "foo:bar", localName: "fool")
	var count = foos?.count
	let node = foos?.first
	let name = node?.nodeName
	let localName = node?.localName
	let prefix = node?.prefix
	let nuri = node?.namespaceURI

	print("“foo:bar”命名空间有以下\(count!) 个元素：\n")
	print("节点名称： \(name!)\n")
	print("本地名称： \(localName!)\n")
	print("前缀： \(prefix!)\n")
	print("命名空间唯一资源路径URI：\(nuri!)\n")

	let children = node?.childNodes

	count = children?.count

	let a = node?.firstChild
	let b = node?.lastChild

	let na = a?.nodeName
	let nb = b?.nodeName
	let va = a?.nodeValue
	let vb = b?.nodeValue

	print("该节点还包含 \(count!) 个子节点。\n")
	print("第一个子节点名称为“\(na!)”，内容值为“\(va!)”。\n")
	print("最后一个子节点名称为“\(nb!)”，内容值为“\(vb!)”。\n")
}

showNamespaces()
```

## XPath

XPath 的官方名称为 XML 路径语言，是用于在 XML文档中筛选节点的一种查询语言。此外，XPath 还可以用于在 XML 文档内用于计算变量值（比如字符串、数值或者逻辑布尔变量值）。

以下程序演示可以根据您自行输入的路径进行节点访问：

``` Swift
func showXPath(xpath: String) {
    /// 请使用.extract()方法来处理 XPath 请求信息：
	let pathResource = xDoc?.extract(path: xpath)
	print("XPath路径： '\(xpath)':\n\(pathResource!)\n")
}

showXPath(xpath: "/rss/channel/item")
showXPath(xpath: "/rss/channel/title/@attribute1")
showXPath(xpath: "/rss/channel/link/text()")
showXPath(xpath: "/rss/channel/item[2]/deeper/foo:bar")
```
