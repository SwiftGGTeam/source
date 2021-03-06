title: "Beta 2 更新：Swift 2.1 Playground 使用值放置方法"
date: 2015-12-16
tags: [Swift 入门]
categories: [Erica Sadun]
permalink: new-to-beta-2-swift-2-1-playground-value-drops-swiftlang

---
> 作者：Erica Sadun，[原文链接](http://ericasadun.com/2015/09/23/new-to-beta-2-swift-2-1-playground-value-drops-swiftlang/)，原文日期：2015-09-23
> 译者：[天才175](http://weibo.com/u/2916092907)；校对：[numbbbbb](http://numbbbbb.com/)；定稿：[numbbbbb](http://numbbbbb.com/)
  







<!--此处开始正文-->

![](https://swift.gg/img/articles/new-to-beta-2-swift-2-1-playground-value-drops-swiftlang/Color-AppScreenSnapz001.png1450226715.996496)

Beta 2 的新特性允许你拖放颜色、图片以及文件。在截图中虽然看不到，但如果你打开文本赋值的历史记录，就会显示文件的文本内容（是我的购物清单，好奇的家伙们）。文本常量的类型为`NSURL`。颜色是`UIColor`，图片是`UIImage`。谢天谢地，希望你们对这些没有任何疑问。

<!--more-->

* 非常有趣的是，你可以在 playground 里拖动它们。所以如果你不小心把一张图片放在了颜色那一行，直接拖到 图片赋值那里就行。
* 你也可以选择将拖动的物品进行复制。
* 双击颜色可以打开颜色选择器（耶！），双击图片可以打开资源文件夹来选取其他资源。双击 URLS 我没发现可以干神马。
* 你不能调整代码中占位符的大小，但是你可以调整它们历史记录界面的大小，和其他值一样。
* 你还不能通过调用`UIColor`来生成颜色预览，比如，`UIColor.blueColor()`就不能生成预览。我发现最简单的方法就是从外部拖进来或者复制/拖另一个颜色，然后用色轮赋值。
* 如果代码中占位符是蓝色，别输入。单击关闭它或者直接用键盘输入内容替换它。蓝色意味着可以选取并且可以被改写。简单吧。

聪明人的做法：**不要把 playground 拖入它自身**。我是认真的，我已经踩过坑了。

![](https://swift.gg/img/articles/new-to-beta-2-swift-2-1-playground-value-drops-swiftlang/Screen-Shot-2015-09-23-at-8.30.41-PM.png1450226717.1296368)

由于文件可以随意复制到资源文件夹。所以：

* 不要通过拖动复制同样的文件两次。Xcode 不喜欢那样。
* 不要指望编辑源文件可以自动同步修改，你需要修改添加后的文件。
* 颜色不能复制到资源里，它们只能是值项。
* 目前有很多事情还不能做。比如，你不能从 Safari “复制” URLS 过来。它们需要进行转义，不然会被当作纯文本。虽然可以期待之后可以拖放的词汇会越来越多，但目前只有颜色、图片和本地文件的 NSURL。
* 如前所述，不要把 playground 拖入它自身。

## 其他新的东西

Swift 关于如何响应引入的 enums，unions, NSNumbers 等有很大的变化。如果你从事大量跨语言编码，值得认真读一读更新说明。

Swift 2.1 现在可以在字符串插值中使用双引号。

> 表达式字符串插值现在可以包含字符串了。比如，“My name is \ (attributes["name"]!)” 现在是有效的。（14050788）

编译器性能有一些提升。没有任何依赖的项（即标记为私有的）不会再触发其他文件的重编译。

更加宽泛的函数类型。你可以这样赋值了，从`任何类型->Int 闭包`到`字符串->任何变量`。这种方式到底好还是不好，我仍在思考中。

> 现在支持函数类型的转换，展现了函数结果类型的协变和函数参数类型的逆变。比如，现在这样的函数类型赋值方式是合法的，从`任何类型->Int 闭包`到`字符串->任何变量`。（19517003）

下面，抛开 playground, 对于我来说，有一个最重大的改变。那就是 map 闭包（\_->\_ 是不是很眼熟？）的错误提示“更加有用了”。我都等不及要试一试了!
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。