# Perfect-Markdown 

该项目提供了在Swift中直接从Markdown文本生成HTML的方法

该软件使用SPM进行编译和测试，本软件也是[Perfect](https://github.com/PerfectlySoft/Perfect)项目的一部分，但也可以独立使用。

请确保您已经安装并激活了最新版本的 Swift 4.0 tool chain 工具链。

## 致谢

Perfect-Markdown 直接基于  [GerHobbelt 的 "upskirt（超短裙）"](https://github.com/GerHobbelt/upskirt) 项目.


## 使用说明

请首先修改您的 Package.swift 文件增加依存关系：

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-Markdown.git", majorVersion: 3)
```

## 引用库函数

请将下列头文件增加到源代码

``` swift
import PerfectMarkdown
```

## 从 Markdown 文本中获取 HTML 字符串

一旦引用成功，String 类型会增加一个名为 `markdownToHTML` 的扩展属性:

```
let markdown = "# 这是一个 markdown 文档 \n\n## with mojo 🇨🇳 🇨🇦"

guard let html = markdown.markdownToHTML else {
  // 转换失败
}//end guard

print(html)
```

