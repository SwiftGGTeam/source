title: "Feature 框架设计"
date: 2017-05-03
tags: [iOS 开发]
categories: [Natasha The Robot]
permalink: architecting-for-features
keywords: 
custom_title: 
description: 

---
> 作者：Natasha The Robot，[原文链接](https://www.natashatherobot.com/architecting-for-features/)，原文日期：2017-03-12
> 译者：[saitjr](http://www.saitjr.com)；校对：[Crystal Sun](http://www.jianshu.com/users/7a2d2cc38444/latest_articles)；定稿：[CMB](https://github.com/chenmingbiao)
  







<!--此处开始正文-->

几个月前，我在 [iOS Conf SG](http://iosconf.sg/) 上做了名为 **创建 Feature，而不是 App** 的分享（你可以在[这里](https://www.youtube.com/watch?v=lJlyR8chDwo)查看完整分享）。在 WWDC 2016 之后，我清晰地认识到今后的发展不是一个大一统的 App，而应该拥有各种小功能的 Feature。想想 Apple Watch，Widget，可交互的 Notifications，App Search，iMessage 贴纸，地图这些数不胜数的应用。

<!--more-->

![](https://www.natashatherobot.com/wp-content/uploads/Screen-Shot-2017-03-12-at-5.15.38-PM-1024x721.png)

如果处理得当，这里的每个 extension 都会成为你应用中的 feature。比如，Widget 中并不需要包含你应用的全部功能，仅提供最实用的那部分，能让用户快速查看即可（就像「天气」应用一样）。

继续深入，我们会发现，如果 extension 的种类足够多，那用户就可以从 app 中解脱了。他们完全可以在不打开 app 的情况下，直接查看相关信息（比如之前提到的「天气」）。虽然还是在使用你的服务，但不再是传统的「点击打开 app」了。

这完全重新定义了 app 的含义。所以，现在 app 不再是真正意义上的应用了，它变成了在合适时间，给用户提供相关信息的东西。这种认知转变，成功规避掉了那些需要依赖于「打开」的 app（比如，内置广告），而且还提供了许多全新的交互方式，树立更为强大的品牌意识。现在，app 能真的被称为「无处不在」了！不再是那个角落中期待被点击的图标了。

这也是为什么在 [try! Swift Tokyo](https://www.tryswift.co/tokyo/en) 大会上，我专注于为 [try! Swift app](https://github.com/tryswift/trySwiftAppFinal) 添加 feature 而不是开发新功能点的原因。

其中的关键是将数据层与 iOS app 分离，包括 Realm。Watch app，Apple TV app（说不定之后会开发），iOS app，iOS extension（如 Widget 和可交互的 Notification） 均会共用数据层。

不过，分离要比想象中复杂。我推荐先从 Basem Emra 的「[用 Carthage 与 CocoaPods 打造跨 iOS，watchOS，tvOS 平台的 Swift 框架](http://basememara.com/creating-cross-platform-swift-frameworks-ios-watchos-tvos-via-carthage-cocoapods/)」入门。

我自己还没独立做过 CocoaPods，更别说还是要依赖 Realm 的，所以我花了很多时间收集梳理。感谢带我入门的 [@aaalveee](https://twitter.com/aaalveee)，让 pod 能在 extension 之间工作的 [@k_katsumi](https://twitter.com/k_katsumi)，帮我集成 [Realm Mobile Platform](https://realm.io/products/realm-mobile-platform/) 的 [@TimOliverAU](https://twitter.com/TimOliverAU)（当然，还有些后续处理）。

虽然过程很困难（到现在都还有些工作没做），但是能有这样的结果我已经非常开心了。你可以在 [GitHub](https://github.com/tryswift/trySwiftData) 上看到 **trySwiftData** 这个框架。框架中最让我满意的地方，是格式化数据的这部分代码，比如，每个话题的标题为演讲者 + 描述，以前处理这个的代码就重复散落在 iOS 和 watchOS 上。我们甚至可以使用同一个 pod 添加有时间限制的 Widget。

期待下一次 try! Swift in NYC 能支持更多的 feature，包括可交互的 Notifications 和 App Search 🚀。
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。