title: "NSPredicate"
date: 2019-11-19
tags: [Objective-C, NSPredicate]
categories: [swiftjectivec]
permalink: nspredicate-objective-c
keywords: predicate, collection
custom_title: "NSPredicate"

---
> 作者：Jordan Morgan，[原文链接](https://www.swiftjectivec.com/nspredicate-objective-c/)，原文日期：2018-05-18
> 译者：[石榴](https://github.com/alejx)；校对：[numbbbbb](http://numbbbbb.com/)，[Nemocdz](https://nemocdz.github.io/)；定稿：[Pancf](https://github.com/Pancf)
  








<!--此处开始正文-->

Swift 刚出现的时候，我们因它比 Objective-C 简洁而着迷。接着它很快打开了面向协议编程的大门。并且，让我们忘掉引用类型和类，还有很多。

确实，这些东西都是很棒的工具，都有优秀的用例。但我感觉它们经常被捧作银弹，在决定架构时缺乏足够的考虑。

<!--more-->

因此在 2018 年，技术博客中充斥着各种 Swift 黑魔法（我的博客也不例外🤷🏻‍♂️），会议演讲也都在讨论 Swift 的函数式编程未来（没错，我也做了这种演讲🙋🏻‍♂️）。

所有人都对在 Swift 中使用集合感到激动，**但是**我们从 iOS 3 开始就可以用 Objective-C 来做*相似*的事了。所以今天我会讨论 `NSPredicate` 的威力，以及如何用🦖筛选集合。

有必要提一下：我们最近看到了一些开发者一开始学了 Swift，后来又得回去维护 Objective-C 的代码。如果说的就是你，那你很可能正在发愁如何优雅地在 Objective-C 中处理集合。

这里讲的东西可能对你有用。

## 用例

近几年来，Objective-C 的集合有了长足的进步。还在几年以前，我们还必须教这愚蠢的编译器：

```Objective-C
NSString *aString = (NSString *)[anArray indexOfObject:0];
```

感谢老天、库比提诺[^1]和朋友们©终于用类型擦除的方式添加了泛型。这是一个很大的进步：

[^1]: 译者注：Cupertino, CA，苹果总部所在城市。

```Objective-C
NSArray *anArray = @[@"Sup"];
NSString *aString = [anArray firstObject];
```

但无论是不是泛型，我们经常通过与下面类似的方法与 Objective-C 集合中的内容交互：

```Objective-C
for (NSString *str in anArray)
{
    if ([str isEqualToString:@"The Key"])
    {
        // 做些什么
    }
}
```
很多情况下，这样写是可以接受的。但是当需求越来越复杂，关系更加多种多样，代码就会变得不确定。如果你遵从代码更少更稳定更容易维护的观念，那么这种简单的查询集合操作也可能成为困扰。

Predicate 可以改善这个状况。不是要在代码中耍些小聪明，而是写出简洁和实用的代码。

## 概览
`NSPredicate` 的核心用途是限制或定义对内存中的数据过滤，或进行取回时的参数。当它和 Core Data 一起使用的时候才会如虎添翼。它和 SQL 很像，只不过没那么糟糕\*。

> 开个玩笑，只是我对基于集合的操作都无感🙃。

你给它提供逻辑条件，然后就会返回符合条件的东西。这意味着它可以提供基础比较、复合 predicate、KeyPath 集合查询、子查询、合计以及更多的支持。

因为它用来筛选集合，它可以获得 Foundation 类的原生支持。使用可变版本时支持用结果直接修改，不可变版本会返回一个新实例：

```Objective-C
// 修改原数组
[mutableArray filterUsingPredicate:/*NSPredicate*/];

// 返回新的数组
[mutableArray filteredArrayUsingPredicate:/*NSPredicate*/];
```

虽然 predicate 可以从 `NSExpression`、`NSCompoundPredicate` 或 `NSComparsionPredicate` 中实例化，但它还可以用一个字符串的语法生成。这和可视化格式语言类似，我们可以用它定义排版约束。

在这里我们主要关注能用字符串语法生成的能力。

## 配置

为了更好的说明，文章的剩余部分以下面的代码为前提。

```Objective-C
// 伪代码
Person:NSObject
Identifier:NSString
Name:NSString
PayGrade:NSNumber

// 某个含有 Person 实例的属性
NSArray *employees
```

## 查询⚡️
本文剩下都在用直接的例子来介绍如何用字符串格式语法来配置查询。

我们可以从一个简单的搜索的情景开始。先假设我们有一个含有表示 `Person` 对象标识符的数组：

```Objective-C
{
    @"erersdg32453tr",  
    @"dfs8rw093jrkls",  
    // etc
}
```

现在，我们想通过这些识别符从一个现存的 `Person` 数组中获取 `Person` 对象。可以使用一个双层嵌套的 `for` 循环来解决这个问题：

```Objective-C
// 假设 "employees" 是一个存有 Person 对象的数组
NSArray  *morningEventAttendees = @[/*上面的人的识别符*/];
NSMutableArray  *peopleAttendingMorningEvent = [NSMutableArray new];

for (NSString *userID in morningEventAttendees)  
{  
    for (Person *person in employees)  
    {  
        if ([person.identifier isEqualToString:userID])  
        {  
            [peopleAttending addObject:person];  
        }  
    }  
}

// 现在 peopleAttendingMorningEvent 里面就有我们想要的东西了
```

我们也可以使用 predicate 来达到完全一样的效果：

```Objective-C
NSPredicate *morningAttendees = [NSPredicate predicateWithFormat:@"SELF.identifier IN %@", peopleAttendingMorningEvent];

NSArray *peopleAttendingMorningEvent = [employees filteredArrayUsingPredicate:morningAttendees];
```

💫。

Predicate 的语法允许我们使用 SELF，它在这里发挥了很大的作用。它表示数组里正在被操作的对象，在这里就是 `Person` 对象。

> 另一个额外的好处是我们不用把数组定义成可变的了。

正是因为这个原因，我们可以访问与 SELF 所表示对象关联的 KeyPath。在上面的代码中，引用了 `identifier` 属性。

如果你喜欢的话，任何 KeyPath 可以用放在 "%K" 位置的变量来表示。这个版本和上面的效果一样：

```Objective-C
[NSPredicate predicateWithFormat:@"SELF.%K IN %@", @"identifier", peopleAttendingMorningEvent];
```

## 复合 Predicate

合并多个比较很简单。假设我们还需要像上面一样找到所有参加活动的人，但还要满足他们的工资水平在 50000 到 60000 之间。

如果使用传统的方法，我们的 if 语句只会越写越长：

```Objective-C
// 和上面的代码一样
if ([person.identifier isEqualToString:userID] && (person.paygrade.integerValue >= 5 && person.paygrade.integerValue <= 10))  
{  
    [peopleAttending addObject:person];  
}
```

但使用一个重构过的 predicate 可以让我们用一种更符合语言习惯的方式来解决问题：

```Objective-C
NSPredicate *morningAttendees = [NSPredicate predicateWithFormat:@"SELF.identifier IN %@ && SELF.paygrade.integerValue BETWEEN {50000, 60000}", peopleAttendingMorningEvent];
```

它允许用不同的操作符表示同样的作用，可以根据你的偏好来提升可读性。比如：
- "&&" 或 "AND"
- "||" 或 "OR"
- "!" 或 "NOT"

如你所想，它们经常会在基本比较操作之间出现，聚合在一个 predicate 里。

## 字符串比较

我们经常会处理一些基于字符串比较的匹配。大家都知道 Objective-C 对冗余代码的无止尽追求，在处理 NSString 的时候也丝毫不减：

```Objective-C
NSString *name = @"Jordan";
name = [name stringByAppendingString:[NSString stringWithFormat:@"%@ %@", @"Wesley", @"Morgan"]];
```

……而 Swift 则一边偷笑，一边低调地把字符串们连接起来。幸亏我们在用 `NSPredicate` 来比较字符串时不会写出上面那么冗余的代码。

```Objective-C
// 假设 mutablePersonAr 是一个 Person 数组，里面有 "Karl" 和 "Jordan"
NSPredicate *namesStartingWithK = [NSPredicate predicateWithFormat:@"SELF.name BEGINSWITH 'K'"];
// 现在只有 Karl 了
[mutablePersonAr filterUsingPredicate:namesStartingWithK];
```

实际上任何比较都可以用 predicate 语法中的 `CONTAINS`、`BEGINSWITH`、`ENDSWITH` 和 `LIKE` 来实现：

```Objective-C
// 假设 mutablePersonAr 是一个 Person 数组，里面有 "Karl" 和 "Kathryn"
NSPredicate *namesStartingWithK = [NSPredicate predicateWithFormat:@"SELF.name LIKE 'Kar*'"];

// 现在只有 Karl 了
[mutablePersonAr filterUsingPredicate:namesStartingWithK];
```

> 你可能已经注意到上面的星号了；和很多的 DSL 一样，这个星号代表一个通配符。

当你在一个查询里结合多个比较运算符时，这种简洁用法的重要性就会体现出来了：

```Objective-C
NSString *predicateFormat = @"(SELF.name LIKE 'Kar*') AND (SELF.paygrade.intValue >= 10)";

NSPredicate *namesStringWithK = [NSPredicate predicateWithFormat:predicateFormat];

// 现在只有 Karl 了
[mutablePersonAr filterUsingPredicate:namesStartingWithK];
```

更进一步，它还支持用 `MATCHES` 语法实现 NSPredicate 的类 SQL 语法与正则表达式混用：

```Objective-C
[NSPredicate predicateWithFormat:@"SELF.phoneNumber MATCHES %@", phoneNumberRegex];
```

然而是时候该指出一点，predicate 语法十分严格。它就是一个字符串。除非你是 Mavis Beacon[^2], 否则你总会一遍又一遍地不小心打错字。

[^2]: 译者注：*Mavis Beacon Teaches Typing*，一款在 1987 年发售的教盲打的软件。

好消息是你会很快的发现问题 — 运行时的异常在等着你。我们获得了能力和灵活性，但在某种程度上失去了静态检查的安全性。

为了说明这一点，这段从上面代码稍微修改而来的代码会导致崩溃。你能看出来是为什么吗？

```Objective-C
NSString *predicateFormat = @"SELF.name LIKE 'Kar*') AND (SELF.paygrade.intValue >= 10)"

NSPredicate *namesStartingWithK = [NSPredicate predicateWithFormat:predicateFormat];

// 现在只有 Karl 了 
[mutablePersonAr filterUsingPredicate:namesStartingWithK];
```

为了减轻这些问题，我经常把 predicate 和 `NSStringFromSelector()` 结合在一起用，以此应对错别字和为以后的重构提供多一层安全保障。

```Objective-C
NSString *predicateFormat = @"(SELF.%@ LIKE 'Kar*') AND (SELF.paygrade.intValue >= 10)"

NSString *kpName = NSStringFromSelector(@selector(identifier));  
NSString *kpPaygrade = NSStringFromSelector(@selector(paygrade));

NSPredicate *namesStartingWithK = [NSPredicate predicateWithFormat:predicateFormat, kpName, kpPaygrade];

// 现在只有 Karl 了
[mutablePersonAr filterUsingPredicate:namesStartingWithK];
```

有点复杂了？确实。更安全了？毫无疑问。

## KeyPath 集合查询

由于基于 KeyPath 的用法，`NSPredicate` 拥有一全套工具去操作它们，以提供一个更好的搜索。考虑下面的代码：

```Objective-C
// 假设一个 Person 对象现在有一个下面的属性：
// NSArray *previousPay

// 找到所有满足过去工资的平均值大于 10 的人
NSString *predicateFormat = @"SELF.previousPay.@avg.doubleValue > 10";
NSPredicate *previousPayOverTen = [NSPredicate predicateWithFormat:predicateFormat];

// 所有过去工资的平均值大于 10 的人
[mutablePersonAr filterUsingPredicate:previousPayOverTen];
```

你可以把 `@avg` 换成：  
* `@sum`
* `@max`
* `@min`
* `@count`

想象下如果不使用 predicate 情况下完成同样的工作，就不得不写大量尽管很简单的代码。你可以开始将这些技巧用在你日常的工具链里。

## 对数组的深究

和 KeyPath 查询很像，predicate 也支持以更细的维度检查数组：
- `array[FIRST]`
- `array[LAST]`
- `array[SIZE]`
- `array[index]`

应用在上面的代码样例上，我们就可以这样查询：

```Obejective-C
// 找到所有过去有三份不同工资的人
NSString *predicateFormat = @"previousPay[SIZE] == 3";

NSPredicate *threePreviousSalaries = [NSPredicate predicateWithFormat:predicateFormat];

// 这些 Person 对象过去有三份不同的工资
[mutablePersonAr filterUsingPredicate:threePreviousSalaries];
```

和在上面提到的一样，我们也可以应用多个条件：

```Objective-C 
// 找到所有过去有三个不同的工资以及第一份工资大于 8 的人
NSString *predicateFormat = @"(previousPay[SIZE] == 3) AND (previousPay[FIRST].intValue > 8)";

NSPredicate *predicate = [NSPredicate predicateWithFormat:predicateFormat];
[mutablePersonAr filterUsingPredicate:predicate];
```

更加深入，你可以使用下面的操作符来实现更复杂的操作：
- `@distinctUnionOfArrays`
- `@unionOfArrays`
- `@unionOfObjects`
- `@distinctUnionOfObjects`

假设我们有一个含有 `Person` 对象的数组，我们需要的是找出在所有数组中识别符不同的 `Person` 实例：

```Objective-C
// 假设 p1/2/3/4 都是 Person 对象
NSArray  *> *previousEmployees = @[@[p1],@[p2,p1,p2],@[p1],@[p4,p2],@[p4],@[p4],@[p1]];

// 获取所有不同的 ID
NSArray *unqiuePreviousEmployeeIDs = [previousEmployees valueForKeyPath:@"@distinctUnionOfObjects.identifier"];

// 现在数组里应该只含有不同的 ID
```

厉害吧！

还有更好玩的呢，还支持子查询：

```Objective-C
// 假设 Person 对象有了一个新的属性表示他们的队伍：
// NSArray  *team;
  
// 从雇员数组中找出这样的人，他们的团队中有人满足这个条件：没有历史工资数据并且工资大于 1
NSString *predicateFormat = @"SUBQUERY(team, $teamMember, $teamMember.paygrade.intValue > 1 AND $teamMember.previousPay == nil).@count > 0";

NSPredicate *predicate = [NSPredicate predicateWithFormat:predicateFormat];
[employeeAr filterUsingPredicate:predicate];
```

当你发现你需要在一个含有对象的数组里搜索，而这些对象含有的属性本身就是一个集合的时候，子查询十分有用。所以在上面的例子里，我们有一个 `Person` 对象的数组，并且查询它的 `teamMember` 数组。

## 便捷才是关键[^3]

[^3]: 译者注：此处原作者用了双关。原文是 "Convenience is Key(Path)"，既有便捷是关键的意思，又在暗指这里的关键其实是 Key Path。

尽管 `NSPredicate` 是为了搜索而设计出来的，但如果你不把它用在和原本设计*稍微*偏离的地方那它就不是 Objective-C 了。这里也不例外。

当你想到 predicate，你想到的是从一个集合里筛选 — 也就是说它的返回值（或更改过的原来数组）还含有相同的东西。

但是也可以让他们含有*不*同的东西。其实我们在之前的代码中已经这样操作过了。上面的二维数组被用来返回一个识别符的数组 — `NSString` 实例。KeyPath 让这些变得可能。

这有一个更直接的例子：

```Objective-C
// 我们得到一个长度大于 10 的识别符字符串的数组
NSString *predicateFormat = @"SELF.identifier.length > 10";
NSPredicate *predicate = [NSPredicate predicateWithFormat:predicateFormat];
NSArray  *longEmployeeIDs = [[employeeArray filteredArrayUsingPredicate:predicate] valueForKey:@"identifier"];

// 现在 longEmployeeIDs 已经不含有 Person 对象了，只有字符串
```

## 总结

马上在 Objective-C 的集合里使用这些语法糖，这样就可以不使用嵌套循环从一个特定的子集中提取数据。使用 `NSPredicate` 可以让眼睛轻松很多。

虽然 Swift 从语言级别支持对集合进行切片操作，但使用创建的 NSPredicate 对象来解决相同的问题也不难。如果你发现你在维护一个成熟的代码库，或是需要用上古时代 Objective-C 的新项目，随心所欲的使用 predicate 吧。

下次见吧✌️。
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。