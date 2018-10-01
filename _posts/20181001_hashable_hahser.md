title: "Hashable / Hasher"
date: 2018-10-01
tags: [Swift, NSHipster]
categories: [Swift, NSHipster]
permalink: hashable_hahser
keywords: Swift,Hashable,Hasher,NSHipster
custom_title: Hashable / Hasher2
description: 本文介绍了 Hashable 和相关的新类型 Hasher

---
> 作者：Matt，[原文链接](https://nshipster.com/hashable/)，原文日期：2018-08-13
> 译者：[Damonwong](https://github.com/Damonvvong)；校对：[Lision](https://lision.me/)，[Yousanflics](http://blog.yousanflics.com.cn)；定稿：[Forelax](http://forelax.space)
  







<!--此处开始正文-->

当你在苹果商店预约天才吧服务后，相关工作人员会帮你登记并且安排特定的服务时间，在被带到座位上之后，工作人员会记录你的身份信息并添加到服务队列当中。

根据一份来自某位前零售店员工的报告表示，对于顾客的描述有着严格的指导方针。他们的外貌特征如：年龄、性别、种族、身高都没有被使用 —— 甚至连头发的颜色都没有被使用。而是通过顾客的着装来描述，例如“黑色的高领毛衣，牛仔裤和眼镜”

这种描述顾客的方式和编程中的哈希函数有很多共同之处。同许多优秀的哈希函数一样，它是连续和易计算的，可用于快速找到你正在寻找的内容（或者人）。这比使用队列要好多了，我想你一定会同意的。

这周我们的主题是 `Hashable` 和相关的新类型 `Hasher`。它们共同组成了 **Swift** 最受喜爱的两个集合类 `Dictionary` 和 `Set` 的基础功能。

<!--more-->

假设你有一个可以比较相等性的对象**[列表](https://en.wikipedia.org/wiki/List_%28abstract_data_type%29)**。要在这个**列表**中找到一个特定的对象，你需要遍历这个**列表**的元素，直到找到匹配项为止。随着你向**列表**中添加更多的元素时，需要找到其中任何一个元素所需的平均时间是线性级的(`O(n)`)。

如果将这些对象存储在一个**[集合](https://en.wikipedia.org/wiki/Set_%28abstract_data_type%29)**中，理论上可以在常量级时间(`O(1)`)内找到它们中的任何一个 - 也就是说，在一个包含 10 个元素的**集合**中查找或在一个包含 10000<sup>\*</sup> 个元素的**集合**中查找所需的时间是一样的。这是怎么回事呢？因为**集合**不是按顺序存储对象的，而是将对象内容计算的<dfn>哈希值</dfn>作为索引存储。当在**集合**中查找对象时，可以使用相同的哈希函数计算新的哈希值然后查找对象存储位置。

\* 如果两个不同的对象具有相同的哈希值时，会产生哈希冲突。当发生哈希冲突时，它们将存储在该地址对应的列表中。对象之间发生冲突的概率越高，哈希集合的性能就会更加线性增长。

## Hashable

在 **Swift** 中，`Array` 为列表提供了标准的接⼝，`Set` 为集合提供了标准的接⼝。如果要将对象存储到 `Set` 中，就要遵循 `Hashable` 协议及其扩展协议 `Equatable`。**Swift** 的标准映射接口 `Dictionary` 对它的关联类型 `Key` 也需要遵循 `Hashable` 协议及其扩展协议。

在 **Swift** 之前的版本中，为了让自定义类型能支持 `Set` 或 `Dictionary` 存储需要写⼤量的 [样板代码](https://nshipster.com/swift-gyb/)。

以下面的 `Color` 类型为例，`Color` 使⽤了 8 位整型值来表示红，绿，蓝色值:

```Swift
struct Color {
    let red: UInt8
    let green: UInt8
    let blue: UInt8
}
```

要符合 `Equatable` 的要求，你需要提供一个 == 操作符的实现。要符合 `Hashable` 的要求，你需要提供⼀个名为 `hashValue` 的计算属性:

```Swift
// Swift < 4.1
extension Color: Equatable {
    static func ==(lhs: Color, rhs: Color) -> Bool {
        return lhs.red == rhs.red &&
               lhs.green == rhs.green &&
               lhs.blue == rhs.blue
    }
}

extension Color: Hashable {
    var hashValue: Int {
        return self.red.hashValue ^
               self.green.hashValue ^
               self.blue.hashValue
    }
}
```

对于大多数开发者⽽⾔，实现 `Hashable` 只是为了能尽快让要做的事情步入正轨，因此他们会对所有的存储属性使⽤[异或](https://en.wikipedia.org/wiki/Exclusive_or)操作，并在某一天调用它。

然⽽这种实现的一个缺陷是高哈希冲突率。由于异或操作满⾜[交换率](https://en.wikipedia.org/wiki/Commutative_property)，像⻘色和⻩色这样不同的颜色也会发⽣哈希冲突:

```Swift
// Swift < 4.2
let cyan = Color(red: 0x00, green: 0xFF, blue: 0xFF)
let yellow = Color(red: 0xFF, green: 0xFF, blue: 0x00)

cyan.hashValue == yellow.hashValue // true, collision
```

大多数时候这样做不会出问题；现代计算机已经足够强大以至于你很难意识到性能的衰减，除⾮你的实现细节存在⼤量问题。

但这并不是说这些细节⽆关紧要 —— 它们往往极其重要。稍后会详细介绍。

## 自动合成 Hashable 实现

从 **Swift 4.1** 开始，如果某个类型在声明时遵循了 `Equatable` 和 `Hashable` 协议并且它的成员变量同时也满足了这些协议，编译器会为其自动合成 `Equatable` 和 `Hashable` 的实现。

除了大大的提高了开发人员的开发效率以外，还可以大幅减少代码的数量。比如，我们之前 `Color` 的例子 —— 现在是最开始代码量的 1/3 :

```Swift
// Swift >= 4.1
struct Color: Hashable {
    let red: UInt8
    let green: UInt8
    let blue: UInt8
}
```

尽管对语言进行了明显的改进，但还是有一些实现细节有着无法忽视的问题。

在 **Swift Evolution** 提案 [SE-0185: 合成 `Equatable` 和 `Hashable` 的实现](https://github.com/apple/swift-evolution/blob/master/proposals/0185-synthesize-equatable-hashable.md) 中， [Tony Allevato](https://github.com/allevato) 给哈希函数提供了这个注释: 

> 哈希函数的选择应该作为实现细节，而不是设计中的固定部分；因此，使用者不应该依赖于编译器自动生成的 Hashable 函数的具体特征。最可能的实现是在每个成员的哈希值上调用标准库中的 `_mixInt` 函数，然后将他们异或组合（^），如同目前 `Collection` 类型的哈希方式一样。

幸运的是，**Swift** 不需要多久就能解决这个问题。我们将在下一个版本得到答案:

## Hasher

**Swift 4.2** 通过引入 `Hasher` 类型并采用新的通用哈希函数进一步优化 `Hashable`

在 **Swift Evolution** 提案 [SE-0206: Hashable 增强](https://github.com/apple/swift-evolution/blob/master/proposals/0206-hashable-enhancements.md) 中：

> 使用一个好的哈希函数时，简单的查找，插入，删除操作都只需要常量级时间即可完成。然而，如果没有为当前数据选择一个合适的哈希函数，这些操作的预期时间就会和哈希表中存储的数据数量成正比。

正如 [Karoy Lorentey](https://github.com/lorentey) 和 [Vincent Esche](https://github.com/regexident) 所指出的那样，`Set` 和 `Dictionary` 等基于哈希的集合主要特点是它们能够在常量级时间内查找值。如果哈希函数不能产生一个均匀的值分布，这些集合实际上就变成了链表。

**Swift 4.2** 中的哈希函数是基于伪随机函数族 [SipHash](https://en.wikipedia.org/wiki/SipHash) 实现的，特别是 [SipHash-1-3 and SipHash-2-4](https://github.com/apple/swift/blob/master/stdlib/public/core/SipHash.swift)，每个消息块有 1 或 2 轮哈希，分别有 3 或 4 轮最后确定。

现在，如果你要自定义类型实现 `Hashable` 的方式，可以重写 `hash(into:)` 方法而不是 `hashValue`。`hash(into:)` 通过传递了一个 `Hasher` 引用对象，然后通过这个对象调用 `combine(_:)` 来添加类型的必要状态信息。

```Swift
// Swift >= 4.2
struct Color: Hashable {
    let red: UInt8
    let green: UInt8
    let blue: UInt8

    // Synthesized by compiler
    func hash(into hasher: inout Hasher) {
        hasher.combine(self.red)
        hasher.combine(self.green)
        hasher.combine(self.blue)
    }

    // Default implementation from protocol extension
    var hashValue: Int {
        var hasher = Hasher()
        self.hash(into: &hasher)
        return hasher.finalize()
    }
}
```

通过抽象隔离底层的位操作细节，开发人员可以利用 **Swift** 内置的哈希函数，这样可以避免再现我们原有的基于异或实现的冲突：

```Swift
// Swift >= 4.2
let cyan = Color(red: 0x00, green: 0xFF, blue: 0xFF)
let yellow = Color(red: 0xFF, green: 0xFF, blue: 0x00)

cyan.hashValue == yellow.hashValue // false, no collision
```

### 自定义哈希函数

默认情况下，Swift 使用通用的哈希函数将字节序列缩减为一个整数。

但是，你可以使用你项目中自定义的哈希函数来改进这个缩减的问题。比如，如果你正在编写一个程序来玩国际象棋或者棋盘游戏，你可以使用 [Zobrist hashing](https://en.wikipedia.org/wiki/Zobrist_hashing) 来快速的存储游戏的状态。

### 避免哈希泛滥(Hash-Flooding)

选择像 **SipHash** 这样的加密算法有助于防止哈希泛滥的 **DoS** 攻击，这种攻击会尝试生成哈希冲突，并试图强制实施哈希数据结构最坏的情况，最终导致程序慢下来。[这在 2010 年初引发了一系列的网络问题](https://arstechnica.com/information-technology/2011/12/huge-portions-of-web-vulnerable-to-hashing-denial-of-service-attack/)。

为了使事情变的更加安全，`Hasher` 会在每次启动应用程序时生成一个随机种子值，使得哈希值更难以预测。

> 你不应该依赖特定的哈希值或者在执行中保存它们。在极少数情况下，你确定要这么做的话，可以设置标识符 SWIFT_DETERMINISTIC_HASHING 以禁用随机哈希种子。

编程类比的挑战在于它们通过边界情况规范反社会行为。

当我们能够考虑到攻击者所有可能利用来达到某种险恶目的的情况时，这时能体现出我们优秀工程师的品质 —— 比如哈希泛滥的 DoS 攻击。在现实生活中，这么做我们需要冒着失败的风险去应用这些 AFK（Away From Keyboard）知识。

也就是说...亲爱的读者，在下次您访问当地的苹果零售店时，我不鼓励你和你的朋友协调服饰，以试图在天才吧中制造混乱和不和谐。

请不要这么做。

相反的，希望你有下面的收获：

当你在天才吧等候的时候，远离那些和自己穿同色衬衫的人。这会让每个人做事都变得容易得多。

> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。