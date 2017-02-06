title: "关于 Swift，我不喜欢的几点"
date: 2017-02-06
tags: [Swift 入门]
categories: [KHANLOU]
permalink: my-least-favorite-thing-about-swift
keywords: 
custom_title: 
description: 

---
> 作者：Soroush Khanlou，[原文链接](http://khanlou.com/2016/08/my-least-favorite-thing-about-swift/)，原文日期：2016-08-14
> 译者：[Joy](undefined)；校对：[冬瓜](http://www.desgard.com/)；定稿：[CMB](https://github.com/chenmingbiao)
  







<!--此处开始正文-->

在以前，我已经写过很多[喜欢 Swift 的理由](http://khanlou.com/2016/05/six-months-of-swift/)。但是今天，我想要写的是这门语言不足的地方。这是一个锱铢必较的问题，所以我将举例描述，去指出这门语言做的好的地方，做的不好的地方，以及其前景。

<!--more-->

### 语言内定义 VS 非语言内定义

看一下 `Ruby` 语言的情况

`Ruby` 的 `attr_accessor` 是一种为实例变量定义 `setter` 和 `getter` 的方法。你可以像下面这样使用它

```swift
class Person
	attr_accessor :first_name, :last_name
end

```

乍一看，它像是一种语言特性，与 `Swift` 的 `let` 和 `var` 属性声明方式相似。但是， Ruby 的函数即便没有括号也可以被调起，而且这只是一个被定义在类范围内的函数（在 Swift 中我们将会调起一个静态函数）：

```swift
def self.attr_accessor(*names)
  names.each do |name|
    define_method(name) {instance_variable_get("@#{name}")} # 这是 getter 方法
    define_method("#{name}=") {|arg| instance_variable_set("@#{name}", arg)} # This is the setter
  end
end
```

如果你不能读懂 `Ruby` ，没有关系。它使用了一个名为 `define_method`的函数来为你传递的变量创建一个 `getter` 和 `setter` 方法。在 `Ruby`中，`@first_name` 意味着一个名为 `first_name` 的实例变量。

这是我爱上 `Ruby` 这门语言的原因之一 ，他们首先设计了元数据工具集，去创建有用的语言特性，然后使用这些工具去实现他们想要的语言特性。[ Yehuda Katz explores](http://yehudakatz.com/2010/02/07/the-building-blocks-of-ruby/)讲述了 `Ruby` 是如何在它的 `blocks` 中实现这一想法的。因为 Ruby 的语言特性是通过相同的工具和相同的语言编写而成，并且这门语言所有用户都有权使用，所以，在这门语言的范畴内和相似风格的情况下，用户也可以编写语言特性。

### 可选类型

Swift 的一个核心特性就是它的可选类型。它允许用户定义某个变量是否可以为空。在系统中，被定义为枚举的格式：

```swift
enum Optional<WrappedType> {
	case Some(WrappedType)
	case None
}

```

就像 `attr_accessor` ，这个特性使用了一个 `Swift` 的语言结构来定义自身。这很不错，因为这意味着用户可以使用不同的语义来创建相似的事物，就像这个虚构的 `RemoteLoading` 类型:

```swift
enum RemoteLoading<WrappedType> {
	case Loaded(WrappedType)
	case Pending
}
```

它和 `Optional`（可选类型）有相同的形态，但有着不同的含义。（[在这篇博文中](http://holko.pl/2016/06/09/data-state-as-an-enum/)，Arkadiusz Holko 曾对这个枚举有更进一步的阐述）

然而，在某种程度上，`Swift` 的编译器知道 `Optional` (可选) 类型但却不知道 `RemoteLoading` （远程加载），它可以让你做一些特殊的事情。看一下这些相同的声明：

```swift
let name: Optional<String> = .None
let name: Optional<String> = nil
let name: String? = nil
var name: String?
```

让我们解析下它们的含义。第一条语句是完整的表述（带有类型推断）。你可以使用相同的语法声明你自己的` RemoteLoading `（远程加载）属性。第二条语句使用了 `NilLiteralConvertible` 协议来定义当你把这个值设置为` nil` 的时候所要执行的操作。虽然这种语法对于你自己的类型访问是可以的，但是使用 `RemoteLoading `（远程加载）却显得不是很正确。这是第一个语言特性，使得` C `族语言开发者对Swift有更舒服的感觉，待会我们会再次提到这一点。

第三条和第四条语句，编译器开始使用 `Optional `(可选) 类型来允许我们编写特殊的代码。第三条语句使用了一个 `Optional` (可选)类型的简写 T?。这被称为 语法糖，可以让你使用简单的方式来编写常用的代码。最后一句是另外一块语法糖：如果你定义一个可选类型，但是你不赋给它任何值，那编译器将会推测出它的值应该为` .None` / `nil` （仅仅当他是一个 `var` 变量的时候才成立）。

后面的两条语句不支持自定义类型。语言的 Optional 类型，可以通过语言内存在的结构定义，以特定类型的异常结束，这个异常只有当前类型可以访问。

### 家族

`Swift` 被定义为“像在 C 语言家族中一样”的语言，这个得益于循环和` For `语句。

`Swift `的 `for..in `语法结构是特殊的。任何遵守 SequenceType 的数据结构，都可以通过一个 for..in 循环来遍历。这就意味着，我可以定义自己的类型值，并声明他们是序列化的，就可以用for..in 循环来遍历它了。

虽然 if 语句和 while 循环是通过 BooleanType 类型 [在 Swift 2.2 中这样子工作的](http://khanlou.com/2016/06/falsiness-in-swift/) ，但是这种功能在 Swift 3 已经被移除了。我不能像在 for..in 循环语句中那样子定义自己的布尔类型值然后在 if 语句中使用。

这里有两种基本的方法去定义语言特性，在 Swift 中都有体现到。第一种是创建一个可以用来定义语言特性的元工具；另一种是定义语言特性和语言类型值之间的一种明确和具体的关系。

你可以会对符合 SequenceType 的类型值比符合 BooleanType 的类型值更加有用这个观点提出异议。但是，Swift 3 已经完全的移除了这个特性，所以，你只能承认：你不得不去认为 BooleanType 是如此没有用处以至于会被完全禁止。

### 运算符

在 Swift 中的运算符也值得研究。语言中存在着定义运算符的语法，所有的算术运算符都是在这个语法中被定义的。用户们可以自由的定义自己的运算符，如果你想要创建自己的  [BigInt](https://github.com/lorentey/BigInt) 类型，同时也想要使用标准的算术运算符，这将是非常有用的。

然而` +` 运算符在语言中被定义，三元运算符 `?: `却没有。当你点击 `+ `时，命令跳转到这个运算符的声明处。当你点击三元运算符中的` ? `和` : `的时候，却没有任何反应。如果你想要在你的代码中使用单个的问号和感叹号作为操作符的话，这是做不到的。注意我这里 不是 说在你的代码中使用一个感叹号操作符不是一个好主意。我只是想说，这个操作符已经被特殊对待，硬编码到了编译器，与其他 `C `中定义的操作符一般无二。

在这三个情况中，我都比较了两个东西：一是被标准库用来实现特性的有用语法，一种是特权标准库超越使用者代码的特殊情况。

最好的语法和语法糖是可以被一门语言的作者利用自己的类型和系统不断深入挖掘的。Swift 有时使用`NilLiteralConvertible`，`SequenceType`和`BooleanType`来处理这些情况。这种 `var name: String?` 能够推测出自己的默认属性值（.None）的方式很明显不符合这个条件，因此这是一种不那么给力的语法糖。

我认为另一个值得注意的点是，即使我爱 Ruby 的语法，但是 Ruby 在运算符和 `falsiness `这两个地方却不是很灵活。你可以自行定义已存在运算符的实现方式，但是不能添加一个新的运算符，而且运算符的优先级也是固定的。Swift 在这个方面更灵活。而且，当然，在 Swift 3 之前，Swift 在定义 `falsiness` 方面同样具有更强的灵活性。

### 错误

在某种程度上来说，Swift 的可选类型类似于 C 语言的可控性， Swift 的错误处理也类似于 C 语言的异常处理。Swift 的错误处理引入了一些新的关键词： `do` ,` try` , `throw `,` throws` , `rethrows `, 和 `catch` 。

使用 `throws` 标记的函数和方法可以` return `一个值或者 `throw` 一个 `ErrorType` 。被抛出的错误将会在 catch blocks函数中被捕捉到。在幕后，你可以想象到 Swift 通过可能代表成功或者失败的 `_Result `类型（就像 [antitypical/Result](https://github.com/antitypical/Result)）重写了函数的返回值类型，例如：

```swift
func doThing(with: Property) throws -> Value
```

重写为

```swift
func doThing(withProperty) -> _Result<Value, ErrorType>
```

事实上，这种 _Result 类型并没有被显式定义，而是 [在编译器中被隐式的处理了](https://marc.ttias.be/swift-evolution/2016-08/msg00322.php) )。这对于我们的例子并没有造成太多的不同。）在调用函数的内部，传入成功的值的时候将会通过 try 语句，而发生错误的时候，则会跳入并执行 `catch block`函数。

对比这个和之前的例子，例子中语言特性被定义在语言内部，再加上语法（例如操作符和 SequenceType）和语法糖（例如 Optional），那么这个代码就变的像我们所期待的那样了。相反的，Swift 的错误处理并没有暴露它的内部 _Result 模型，所以用户无法使用或者改变它。

一些情况下使用 Swift 模型来进行错误处理非常合适，例如  [Brad Larson 用来移动机器人手臂的代码](http://www.sunsetlakesoftware.com/2015/06/12/swift-2-error-handling-practice)和 [我的 JSON 解析代码](http://khanlou.com/2016/04/decoding-json/) 。其他情况的话，使用 Result 类型和 flatMap 会更合适。

其他的代码可能依赖异步处理，并想要传递一个 Result 的类型值给`completion block`。苹果的解决方案只能在某些特定的情况下起到作用，给予在错误模型上更大的自由可以帮助缩小这门语言和使用者之间的距离。Result 是很好的，因为它足够灵活，可以在上面玩很多花样。 try / catch 语法并不是很给力，因为它的使用十分严格而且只有一种使用方法。

### 未来

Swift 4 承诺在不久后，将使用异步的语言特性。目前还不清楚将如何实现这些功能，但是 Chris Lattner 已经写了很多关于 Swift 4的东西

> 一类并发：Actors、同步/等待、原子性、内存模型及其它一些相关主题

Swift 的异步的处理机制将会是什么样子的，异步/等待 是我的主要理论。在外行人看来，异步/等待 声明什么时候函数是异步的

```swift
async Task<int> GetIntAsync()
{
    return new Task<int>(() =>
    {
        Thread.Sleep(10000);
        return 1
    });
}

async Task MyMethodAsync()
{
    int result = await GetIntAsync();
    Console.WriteLine(result);
}
```

第一个函数方法， `GetIntAsync` 返回了一个任务，该任务等待一段时间后返回了一个值。因为这个函数返回了一个 `Task` ，所以被标记为 `async`。第二个函数方法，首先调用 `MyMethodAsync` ，使用关键词 `await` 。这通知了整个系统，在完成 `GetIntAsync `任务之前，这个系统可以做其他的事情。而一旦这个任务完成了，这个函数就会恢复控制功能，并在控制台打印。

从这个例子看来，`C#` 的 `Task` 对象看起来很像  [Promise](http://khanlou.com/2016/08/promises-in-swift/) 。此外，任何使用 await 的函数都必须被定义为 async 。编译器可以确保这点。这个解决方案与 Swift 的错误处理模型很相似：被抛出的函数方法必须被捕捉到，而如果没有，那这些函数方法一定也是被标记了 throws 。

它也像错误处理模型一样有着缺陷。在加上新构造和一些关键词之后，更像是语法糖，而不是一个有用的工具。这种构造一部分依赖于在标准库中定义的类型，一部分依赖于编译器定义的语法。

### 属性

属性行为是 Swift 4 可能引入的另一个重大特性。这里是关于属性行为的[拒绝提案](https://github.com/apple/swift-evolution/blob/master/proposals/0030-property-behavior-decls.md)，在 Swift 4中，这一个特性被更密切的关注。

属性行为让你可以对一个属性附加上行为，比如添加 lazy。这个 lazy 属性，举个例子，只有它在被第一次访问时才设置值。但你现在已经可以使用这个特定的行为，这是直接硬编码进 Swift 编译器的。属性行为将使标准库更容易地实现一些行为，同时方便用户去完全自定义行为。

可能这已经是全世界最好的特性了。从一个已经被硬编码进编译器的一个特性开始，然后在这个特性取得一定声望之后，创建一个更通用的框架来允许你通过语言本身定义这个特性。基于这一点，,任何 Swift 的开发者都可以实现类似的功能，精确调整来满足自己的需求。

如果 Swift 的错误模型遵循着相同的路径，Swift 的标准库可能会暴露出一个 Result 类型值，然后任何返回 Result 值的函数，都可以在必要的时候，使用 `do` / `try` /` catch `语法（就像那些可以单个失败的并行、同步事件）。对于那些不需要符合当前可用语法的错误，就像异步错误，用户将可以使用一个共同的 `Result `。这个 `Result `需要很多链，用户可以 `flatMap` 。

异步／等待可以按照同样的方式。定义一个 `Promise` 或 `Task` 协议，并遵守他们，那么任务将是可以等待的（await）。`then `和` flatMap` 在这里是可用的，根据用户的需求，可以来选择对应的语言特性。

### 元编程

我想要更多地去写一些关于元编程的知识。我已经写了[关于 Objective-C 中的元编程](http://genius.com/Soroush-khanlou-metaprogramming-isnt-a-scary-word-not-even-in-objective-c-annotated)，它和我们正在着手做的事情很相似。代码和元代码之间的界限是模糊的。Swift 编译器中的代码是元代码，并且 Swift 本身也是代码。如果定义一个 `operator `函数的实现（就像你使用Ruby一样）就是代码，那么定义一个全新的运算符看起来就像是元代码。

作为一种面向协议的语言，Swift 很独特，可以让我们挖掘这门语言的语法魅力，就像我们用 `BooleanType `和 `SequenceType `所做的一样。我很乐意去看一下这些被扩展的能力。

关键词停止和语法开始或者语法停止和语法糖开始的界限，不是很明确，但是对于使用这门语言编写代码的工程师，应该有能力去使用那些开发标准库的工具。
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。