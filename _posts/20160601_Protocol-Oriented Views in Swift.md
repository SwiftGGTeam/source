title: "用 Swift 编写面向协议的视图"
date: 2016-06-01
tags: [Swift 进阶]
categories: [Natasha The Robot]
permalink: protocol-oriented-views-in-swift
keywords: swift面向协议
custom_title: 
description: Swift 面向协议编程是很多开发者很看重的，那么如何使用 Swift 编写面向协议的视图呢。

---
> 作者：Natasha The Robot，[原文链接](https://www.natashatherobot.com/protocol-oriented-views-in-swift/)，原文日期：2016-05-13
> 译者：[Lanford3_3](http://lanfordcai.github.io)；校对：[numbbbbb](http://numbbbbb.com/)；定稿：[Channe](http://www.jianshu.com/users/7a07113a6597/latest_articles)
  







<!--此处开始正文-->

*[和我一起参加](http://www.tryswiftnyc.com)9 月 1 日 - 9月 2 日在纽约举办的 Swift 社区庆典🎉吧！使用优惠码 NATASHATHEROBOT 可以获得 $100 的折扣！*

我最近做了个 Swift [面向协议编程实践](http://www.slideshare.net/natashatherobot/practial-protocolorientedprogramming)（POP💥） 的演讲。视频还在处理中。另一方面，这是演讲中 POP 视图部分的文本记录，供我和其他任何人作参考！

<!--more-->

## 简单的任务

假设你要写一个由一张图片和一个按钮构成的简单应用，产品经理希望按钮被点击的时候图片会抖动，就像这样：

![1.gif](http://static.zybuluo.com/numbbbbb/ki8ralzlrckq3a09e993bsu7/1.gif)

由于这个动画常常在用户名或者密码输入错误时被用到，所以我们很容易就能[在 StackOverflow 上找到代码](http://stackoverflow.com/questions/3844557/uiview-shake-animation#)（就像每个好的开发者都会做的一样😁）

这个需求最难的地方就是决定实现抖动的代码应该写在哪儿，但这其实也没多难。我写了个 UIImageView 的子类，再给它加上一个 **shake()** 方法就搞定了。

```swift

//  FoodImageView.swift
 
import UIKit
 
class FoodImageView: UIImageView {
    
    // shake() 方法写在这儿
    func shake() {
        let animation = CABasicAnimation(keyPath: "position")
        animation.duration = 0.05
        animation.repeatCount = 5
        animation.autoreverses = true
        animation.fromValue = NSValue(CGPoint: CGPointMake(self.center.x - 4.0, self.center.y))
        animation.toValue = NSValue(CGPoint: CGPointMake(self.center.x + 4.0, self.center.y))
        layer.addAnimation(animation, forKey: "position")
    }
}
```

现在，当用户点击按钮的时候，我只要调用 ImageView 的 shake 方法就行了：

```swift
//  ViewController.swift
 
import UIKit
 
class ViewController: UIViewController {
 
    @IBOutlet weak var foodImageView: FoodImageView!
    
    @IBAction func onShakeButtonTap(sender: AnyObject) {
        // 在这里调用 shake 方法
        foodImageView.shake()
    }
}
```

这并没什么令人激动的。任务完成，现在我可以继续处理别的任务了……感谢 StackOverflow!

## 功能拓展

然而，就像实际开发中会发生的那样，当你认为你搞定了任务，可以继续下一项的时候，设计师跳了出来告诉你他们希望按钮能够和 ImageView 一起抖动……

![2.gif](http://static.zybuluo.com/numbbbbb/juckii2ktoi9eyixvumh1k7u/2.gif)

当然，你可以重复上面的做法--写个 UIButton 的子类，再加个 shake 方法：


```swift
//  ShakeableButton.swift
 
import UIKit
 
class ActionButton: UIButton {
 
    func shake() {
        let animation = CABasicAnimation(keyPath: "position")
        animation.duration = 0.05
        animation.repeatCount = 5
        animation.autoreverses = true
        animation.fromValue = NSValue(CGPoint: CGPointMake(self.center.x - 4.0, self.center.y))
        animation.toValue = NSValue(CGPoint: CGPointMake(self.center.x + 4.0, self.center.y))
        layer.addAnimation(animation, forKey: "position")
    }
 
}
```

现在，当用户点击按钮的时候，你就可以让 ImageView 和按钮一起抖动了：

```swift
//  ViewController.swift
 
class ViewController: UIViewController {
 
    @IBOutlet weak var foodImageView: FoodImageView!
    @IBOutlet weak var actionButton: ActionButton!
    
    @IBAction func onShakeButtonTap(sender: AnyObject) {
        foodImageView.shake()
        actionButton.shake()
    }
}
```

但愿你没这么做……在两个地方重复编写 **shake()** 方法违背了 DRY（don't repeat yourself）原则。如果之后一个设计师又过来表示需要更多或者更少的视图进行抖动，你就不得不在多处修改逻辑，这样当然并不理想。

所以该如何重构呢？

## 通常的处理方式

如果你写过 Objective-C, 你很可能会把 **shake()** 写到一个 UIView 的分类（Category) 中（也就是 Swift 中的拓展 (extension)）：

```swift
//  UIViewExtension.swift
 
import UIKit
 
extension UIView {
    
    func shake() {
        let animation = CABasicAnimation(keyPath: "position")
        animation.duration = 0.05
        animation.repeatCount = 5
        animation.autoreverses = true
        animation.fromValue = NSValue(CGPoint: CGPointMake(self.center.x - 4.0, self.center.y))
        animation.toValue = NSValue(CGPoint: CGPointMake(self.center.x + 4.0, self.center.y))
        layer.addAnimation(animation, forKey: "position")
    }
}
```

现在，UIImageView 和 UIButton（以及其他所有视图）都有了可用的 **shake()** 方法：

```swift
class FoodImageView: UIImageView {
    // 其他自定义写在这儿
}
 
class ActionButton: UIButton {
    // 其他自定义写在这儿
}
 
class ViewController: UIViewController {
 
    @IBOutlet weak var foodImageView: FoodImageView!
    @IBOutlet weak var actionButton: ActionButton!
    
    @IBAction func onShakeButtonTap(sender: AnyObject) {
        foodImageView.shake()
        actionButton.shake()
    }
}
```

然而，你立刻就会发现，在 **FoodImageView** 或者 **ActionButton** 的代码中并没有什么特别的东西表示它们能够抖动。只是因为你写了那个拓展（或分类），你知道有那么一个能实现抖动的方法被放在其中某处。

再进一步说，这种分类模式很容易就会失控。分类容易变成一个垃圾桶，以存放那些你不知道该放到哪里的代码。很快，分类里的东西就太多了，你甚至都不知道一些代码为什么在那儿，又该用在哪儿。你可以从 [为什么分类被认为是不好的](http://www.catehuston.com/blog/2016/02/04/categories-considered-harmful/) 中了解更多。

所以，该怎么做呢……💭

## 用协议（Protocol）来搞定！

你猜对了！Swifty 的解决方案就是用协议！我们能够利用协议拓展的力量来创建一个带有默认 **shake()** 方法实现的 **Shakeable** 协议：

```swift
//  Shakeable.swift
 
import UIKit
 
protocol Shakeable { }

// 你可以只为 UIView 添加 shake 方法！
extension Shakeable where Self: UIView {
    
    // shake 方法的默认实现
    func shake() {
        let animation = CABasicAnimation(keyPath: "position")
        animation.duration = 0.05
        animation.repeatCount = 5
        animation.autoreverses = true
        animation.fromValue = NSValue(CGPoint: CGPointMake(self.center.x - 4.0, self.center.y))
        animation.toValue = NSValue(CGPoint: CGPointMake(self.center.x + 4.0, self.center.y))
        layer.addAnimation(animation, forKey: "position")
    }
}
```

现在，我们只需要让任何确实需要抖动的视图遵从 Shakeable 协议就好了：

```swift
class FoodImageView: UIImageView, Shakeable {
    // 其他自定义写在这儿
}
 
class ActionButton: UIButton, Shakeable {
    // 其他自定义写在这儿
}
 
class ViewController: UIViewController {
 
    @IBOutlet weak var foodImageView: FoodImageView!
    @IBOutlet weak var actionButton: ActionButton!
    
    @IBAction func onShakeButtonTap(sender: AnyObject) {
        foodImageView.shake()
        actionButton.shake()
    }
}
```

这里需要注意的第一点是可读性！仅仅通过 **FoodImageView** 和 **ActionButton** 的类声明，你就能立刻知道它能抖动。

如果设计师跑过来表示希望在抖动的同时 ImageView 能暗淡一点儿，我们也能够利用相同的协议拓展模式添加新的功能，进行超级赞的功能组合。

```swift
// 添加暗淡功能
class FoodImageView: UIImageView, Shakeable, Dimmable {
    // 其他实现写在这儿
}
```

而且，当产品经理不再想让 ImageView 抖动的时候，重构起来也超级简单。只要移除对 Shakeable 协议的遵从就好了！

```swift
class FoodImageView: UIImageView, Dimmable {
    // 其他实现写在这儿
}
```

## 结论

使用协议拓展来构造视图, 你就为你的代码库增加了超级棒的**可读性**，**复用性**和**可维护性**

P.S. 我推荐阅读 [透明视图控制器及背景遮罩](http://www.totem.training/swift-ios-tips-tricks-tutorials-blog/ux-chops-dim-the-lights) 教程以了解更多这种模式的高级应用。


> 译者注，原文评论中有人认为 “面向协议的视图” 并没必要，增加了过多的代码（每个功能都要写个协议）及不必要的代码层次（分类/拓展的话是 类 -> 方法，而协议是 类 -> 协议 -> 方法），一般的需求没必要这样，并提供了一个演讲供参考，演讲大意是避免不必要的层层封装，保持简单实现，代码的未来的拓展什么的自然有维护团队（=，=？）做等等。另外也有其他读者对之进行了反驳，感兴趣可以看看。个人还是支持作者的观点。


> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。