title: "使用 Storyboard 和 segue 实现 View Controller 数据注入"
date: 2016-05-04
tags: [Swift 进阶]
categories: [Natasha The Robot]
permalink: ios-view-controller-data-injection-with-storyboards-and-segues-in-swift
keywords: viewcontroller之间传值,swift storyboard,swift segue
custom_title: 
description: 在 Swift 中怎么实现 viewcontroller 之间传值呢，用 Storyboard 和 segue 就能搞定哦。

---
> 作者：Joe，[原文链接](https://www.natashatherobot.com/ios-view-controller-data-injection-with-storyboards-and-segues-in-swift/)，原文日期：2015-12-28
> 译者：[aaaron7](http://www.jianshu.com/users/9efd08855d3a/)；校对：[walkingway](http://chengway.in/)；定稿：[numbbbbb](http://numbbbbb.com/)
  







<!--此处开始正文-->

自从我之前在[这篇文章](https://www.natashatherobot.com/i-heart-storyboards-nibs/)中公开表达我对 Storyboard 和 Nib 的热爱之后，就一直有很多人问我如何不用自定义的初始化方法来实现不同的 ViewController 之间的数据传递。现在我来分享一下。

<!--more-->

首先声明一点，我的解决办法并不完美，它是基于 ViewController 工作原理的一种变通方案。希望 Apple 今后能够提供更加友好、Swift 风格的 API(即用 Protocol 来替代继承) 来实现更加通用的依赖注入的方法。

其次，我之所以反对使用自定义的初始化方法，是因为我们必须从 UIKit 的 UIViewController 派生出我们的类，它原本就已经有很多初始化方法了。而且如果我们自己做初始化，还得手动去调用父类的初始化方法，因为 UIKit 需要在父类的初始化方法中做一些工作(这部分源代码是看不到的)。

```swift
// 注释来自于苹果的官方文档
/*
      指定初始化方法，如果你创建 UIViewController 的子类，则必须调用其父类方法，即使你并没有使用 NIB（方便期间，默认的 init 方法
      会为你处理这一切，而且针对这些方法的参数都设为 nil）对于指定的 NIB，该文件的所有者代理拥有的类应当为你视图控制器的子类，
      这一过程是通过 view outlet 与 main view 的连接完成的。如果你在初始化方法中传入空的 nib 名，那么 -loadView 
      方法会尝试载入一个与 view controller 同名的 NIB，如果没有对应的 NIB 存在，则你必须在 -view 被执行前调用 -setView: 方法，
      或者覆盖 -loadView 方法来手动指定你的 view
    */
    public init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: NSBundle?)
    public init?(coder aDecoder: NSCoder)
```

当然，他们看上去差不多，并且似乎不会改变，但我并不能控制或者看到 UIKit 内部的实现。我更愿意去试着使用它们，而不是创建一个自己的初始化方法。

这里我打算通过**Optional 的隐式解包**来解决这个问题😱😱😱。

我会用[这个工程来作例子](https://www.natashatherobot.com/protocol-oriented-segue-identifiers-swift/)。它的详情页显示的内容依赖于前一个页面传过来的值，在这里我用一个隐式解包的 Optional 来存储这个值，该值在页面初始化或者完成载入时必须被赋值。

```swift
// Detail View Controller
import UIKit
 
class RedPillViewController: UIViewController {
 
    @IBOutlet weak private var mainLabel: UILabel!
    
    // 隐式解封包!
    var mainText: String!
    
    override func viewDidLoad() {
        super.viewDidLoad()
 
        // 他在这里使用!
        mainLabel.text = mainText
    }
 
}
```

这个例子中，**如果 `mainText` 没有在 `prepareForSegue` 中被赋值**，这个页面就会马上 crash。这就可以帮助开发者快速定位到问题并为这个依赖赋值。

```swift
// 在父类中 / 更为通用的 View Controller
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        
        switch segueIdentifierForSegue(segue) {
            
        case .TheRedPillExperience:
            let redPillVC = segue.destinationViewController as! RedPillViewController
            // 在这里为隐式解包赋值!
            redPillVC.mainText = "😈"
        case .TheBluePillExperience:
            let bluePillVC = segue.destinationViewController as! BluePillViewController
            // 在这里为隐式解包赋值!
            bluePillVC.mainText = "👼"
        }
    }
```


这里的关键在于**确保隐式解包的值在页面的初始化方法中被使用，或者被赋值**。如果初始化之后才用，那页面就不会马上 crash。就像[@justMaku](https://twitter.com/justMaku)在[这里](https://twitter.com/justMaku/status/714116530926125057?ref_src=twsrc%5Etfw) 所指出的那样。

我刚才也提到，这非完美的解决方案，只是一个比额外自定义初始化方法更好的方案。

你可以在这里看到[完整的例子工程](https://github.com/NatashaTheRobot/POSegueIdentifiers)


> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。