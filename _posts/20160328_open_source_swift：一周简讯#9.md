title: "Swift 一周简讯 #9"
date: 2016-03-28 11:20:00
tags: [Swift 开源信息]
categories: [Open Source Swift]
permalink: open-source-swift-weekly-9
keywords: Swift开源
custom_title: 
description: 

---
> 作者：Jesse Squires，[原文链接](http://swiftweekly.github.io/issue-9/)，原文日期：2016/02/11
> 译者：[pmst](http://www.jianshu.com/users/596f2ba91ce9/latest_articles)；校对：[numbbbbb](http://numbbbbb.com/)；定稿：[numbbbbb](http://numbbbbb.com/)
  








<!--此处开始正文-->

欢迎收看 Swift 开源周刊#9！由于周刊期号是从0开始的，这意味着本周已经是我们的第十期拉。🎉感谢亲们长久以来的陪伴！😄本周开源进展似乎略有放缓。甚至 Xcode7.3 beta3 的[发布说明](http://adcdownload.apple.com/Developer_Tools/Xcode_7.3_beta_3/Xcode_7.3_beta_3_Release_Notes.pdf)也略显平淡。这是否意味着我们越来越接近 Xcode7.3 和 Swift2.2 的正式发布呢。

<!--more-->

### Commits 和 pull requests

John Holdsworth 合并了一个 [pull request](https://github.com/apple/swift/pull/1193)，修复了 switch 枚举用例匹配（case）中缺失“Quick Help”的问题。

Dmitri Gribenko 已开始着手启用 [iOS 主机测试](https://github.com/apple/swift/pull/1215)。

来自 IBM 的 Divid Grove 研究[合并](https://github.com/apple/swift-corelibs-libdispatch/pull/43) Swift 的初始化整合，将其覆盖到 libdispatch 构建中，以及做出相关[修改](https://github.com/apple/swift/pull/1212)使得能够同时构建 corelibs-libdispatch 和 corelibs-foundation。 

Brian Gesiak 向 Apple 提交了 CI 要求覆盖 corelibs-xctest，目前已经[采纳](https://twitter.com/modocache/status/697062595396816896)了！😎

[Swift 基准组件](https://github.com/apple/swift/tree/master/benchmark) 目前已经[开源](https://swift.org/blog/swift-benchmark-suite/)！“该组件代码囊括了基准，库和工具，旨在帮助我们在提交代码前跟踪 Swift 性能和捕获性能回归。”

Nadav Roterm [改善](https://github.com/apple/swift/commit/422764545c720f696bf7061513eac30941d39cf4)了 Swift runtime 中的元数据哈希函数（metadata hash function）。

### Proposals

Daniel Dunbar 开始着手准备关于“Package Manager C Language Target Support”的[新提案](https://github.com/apple/swift-evolution/pull/146)。ps：具体请点击[这里](https://github.com/ddunbar/swift-evolution/blob/swiftpm-c-language-targets/proposals/NNNN-swiftpm-c-language-targets.md)。

> 该提案旨在新增一个初始包管理器，支持 C，C++，Objective-C 以及 Objective-C++ 语言。【省略部分文字...】我们希望 Swift 包能够将 C 目标文件囊括进来，且这些目标文件能够直接作为单一包的一部分为 Swift 提供接口。这样使得开发者和 C 之间提供了一种简单的“回退”机制...

该话题早在1月的邮件列表线程就已经存在，不过看起来 Daniel 才刚刚提交了该 pull request。总而言之，该提案是具有实际意义的，不过在 SPM 团队中优先级较低。倘若被采纳，Swift 包管理器有可能取代 [CocoaPods](https://cocoapods.org/) ——  而且我猜测这很快就会发生。😁除此之外，来自 IBM 的 Ricardo  Olivieri 开启了一个关于外部关系和 SPM 的[相关线程](https://lists.swift.org/pipermail/swift-build-dev/Week-of-Mon-20160125/000253.html)。

Joe Groff 的 “Property Behaviors” 提案（[SE-0030](https://github.com/apple/swift-evolution/blob/master/proposals/0030-property-behavior-decls.md)）目前正在[审核](https://lists.swift.org/pipermail/swift-evolution-announce/2016-February/000034.html)中。这在[邮件列表](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003148.html)中曾讨论过一段时间，我认为整个社区对于该提案都显得十分亢奋。我坚信它最终会被接受。

Chris Lattner 提出有关“Remove implicit tuple splat behavior from function applications”的方案（[SE-0029](https://github.com/apple/swift-evolution/blob/master/proposals/0029-remove-implicit-tuple-splat.md)）已经正式被采纳并收录到 Swift3 当中了。当前除可以使用典型的方式调用函数之外，我们还可通过传入 N个参数的元组（N-tuple）作为函数参数列表。不过这个鲜为人知的特性正逐渐从 Swift3 中移除，因为它仅是纯粹的语法糖罢了。

``` swift
// some function
func foo(a: Int, b: Int) { }

// 典型调用
foo(42, b: 17)

// 使用元组方式 译者注：swift 中将被移除
let x = (1, b: 2)
foo(x)
```

有关该提案的大多数反馈都表示“我从未使用过该特性”，因此我猜测这很有可能被废弃掉。不过，Brent Royal-Gordon 已经创建关于这个话题的[讨论](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009596.html)了。

### Mailing lists

Félix Cloutier 创建有关垃圾回收机制的[话题](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009403.html)。[Joe Groff](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009405.html) 和 [Chris Lattner](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009422.html) 阐述了引用计数对分代标记清理的优势和权衡。同时 Joes 分享了当时尝试标记清理的一些历史和惨痛的教训，而 Austen Zheng 则[提醒](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009556.html)我们安卓在这方面的困境。

Jordan Rose [询问](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009451.html)关于 Swift 中对 library 演变的支持。严格意义上来说这并非是一个提案，因此也无法通过正规的 Swift 演变提案流程，不过反馈结果是鼓励该行为的。你能够在[这里](http://jrose-apple.github.io/swift-library-evolution/)找到完整的文档。 

随着来自社区的大量反馈，Dave Abrahams 修改了早前由他创建的“何时使用参数标签（一个新方法）”话题中的想法，同时重新创建了一个新的话题支线继续该话题。倘若你还没有头绪，其实标题已经很好地描述了该想法。下面举个比较恰当的例子：`a.moveFrom(b,to:c)`或`a.move(from:b,to:c)`？显然，完整的讨论更加细致。API 设计准则的一个主要目标就是帮助回答类似这样的问题。
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。