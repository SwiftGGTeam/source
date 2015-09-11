title: 如何在Swift中优雅地使用UIImage  
date: 2015-09-11 
tags: [Swift]  
categories: [Natasha The Robot]  
permalink: A-Beautiful-Solution-to-Non-Optional-UIImage-Named-in-Swift

---
> 作者：Natasha，[原文链接](http://natashatherobot.com/non-optional-uiimage-named-swift/)，原文日期：2015/08/30
> 译者：[小铁匠Linus](http://weibo.com/linusling)；校对：[Prayer](http://www.futantan.com)；定稿：[shanks](http://codebuild.me/)
  








昨天，我抽空看了[Swift in Practice WWDC15 Session](https://developer.apple.com/videos/wwdc/2015/?id=411)的视频，很喜欢其中对 `Image` 命名的处理建议。

这个视频里解决的问题是方法`UIImage:named:`总需要传入硬编码(写死)的字符串参数，然后返回一个可空(optional)的`UIImage`。这就意味着可能会有两种出错的情况：一种是字符串的拼写错误；另一种是对可选的`UIImage`不正确解包。


<!--more-->
一种可以解决这个字符串拼写错误的方式就是构建一个`Image`的常量文件，但是这不能解决出错的第二种情况。在 Swift 中，有个更好的解决方案。可以扩展`UIImage`，把所有的`Image`名字作为枚举类型，然后建立便利构造器来通过枚举构造对应的`Image`。看代码：

```
//  UIImage+Extension.swift
import UIKit

extension UIImage {
    enum AssetIdentifier: String {
        // Image Names of Minions
        case Bob, Dave, Jorge, Jerry, Tim, Kevin, Mark, Phil, Stuart
    }
    
    convenience init!(assetIdentifier: AssetIdentifier) {
        self.init(named: assetIdentifier.rawValue)
    }
}
```

这样，就可以通过以下的方式在任何需要的地方构建`Image`：

```
let minionBobImage = UIImage(assetIdentifier: .Bob)
```

这样的方式是不是很清晰呀，哈哈。首先，使用了漂亮的枚举值，关键是不再需要硬编码的字符串了。并且，枚举的值是自动填充出来的，不必担心拼写错误。其次，`Image`不再是可空的了，因为你可以确保它一定是存在的。

我自己建了个工程测试了一下，工程在 Github 可以下载，地址在[这里](https://github.com/NatashaTheRobot/ImageNamingInSwift)。如果你想知道它如何在一个app中实现的，可以 check out 后看看。

## Update

正如很多读者指出的，有很多开源的第三方库可以将`Image`名字导出成枚举类型。
可以 check out 以下的第三方库后查看：

* [Shark](https://github.com/kaandedeoglu/Shark)

* [SwiftGen](https://github.com/AliSoftware/SwiftGen)

同时，可以在下面添加任何的评论。
