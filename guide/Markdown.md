# Perfect-Markdown 


This project provides a solution to convert markdown text into html presentations.

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project but can also be used as an independent module.

## Acknowledgement

Perfect-Markdown is directly building on  [GerHobbelt's "upskirt"](https://github.com/GerHobbelt/upskirt) project.


## Swift Package Manager

Add dependencies to your Package.swift

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Markdown.git", majorVersion: 3)
```

## Import Perfect Markdown Library

Add the following header to your swift source code:

``` swift
import PerfectMarkdown
```

## Get HTML from Markdown Text

Once imported, a new String extension `markdownToHTML` would be available:

```
let markdown = "# some blah blah blah markdown text \n\n## with mojo 🇨🇳 🇨🇦"

guard let html = markdown.markdownToHTML else {
  // conversion failed
}//end guard

print(html)
```
