title: "推荐给菜鸟的编程书"
date: 2016-08-22
tags: [iOS 开发]
categories: [KHANLOU]
permalink: resources-for-new-programmers
keywords: 编程入门教程,新手编程入门
custom_title: 新手编程入门教程推荐
description: 想学编程的话入门要看哪些编程书籍呢，本文就推荐了一些优质作者和一些不错的新手编程。

---
> 作者：Soroush Khanlou，[原文链接](http://khanlou.com/2016/06/resources-for-new-programmers/)，原文日期：2016-6-27
> 译者：X140Yu；校对：[Crystal Sun](http://www.jianshu.com/users/7a2d2cc38444/latest_articles)；定稿：[CMB](https://github.com/chenmingbiao)
  







<!--此处开始正文-->

有一些刚刚开始编程的人会问我，如何才能从写代码转变成写漂亮的代码，为此，我会推荐一些书，对于刚开始编程的菜鸟来说，这些书非常赞。对于像我这种已经有几年编程经验的老鸟来说，仍能从中学到东西。这些书有很大一部分都是用其他语言如 C、Ruby 或者 Java 写的，读这些书可能是个不小的挑战。还好在大多数情况下，任何编程语言都可以清晰表达编程思想，况且早点培养多语言编程技能也不是一件坏事。

<!--more-->

### Practical Object-Oriented Design in Ruby

*作者 Sandi Metz*

长时间关注我的读者都了解我对 Sandi Metz 的偏爱。我觉得她做的最棒的事情就是给聪明人解释简单的概念，本书也不例外。

她用修自行车来类比面向对象，从只包含一个方法的对象开始完整地实现Fowler在Refactoring一书中介绍的“用多态模式替换条件表达式”。这本书并不是一下扔给你一大堆概念，而是循序渐进由浅入深地进行讲解。

### Design Patterns

*作者 Gamma、Helm、Johnson 和 Vlissides*

这四位作者被大家称为「四剑客」，这本书出版于 1994 年。是第一本介绍常见的设计模式的书，讲述什么时候应该如何使用这些设计模式，附带使用示例代码。我推荐的几本书都是这种类型的，虽然看起来很像教科书，但也能可以读普通书那样快速翻阅，当要使用某个设计模式时，就知道应该跳到哪一章来获取需要的详细知识了。

这本书创作于在桌面应用时代，其中的一些设计模式也是针对那个时代的。比如命令模式，对于菜单中没有功能，使用命令行非常有用。但是这些动作场景在 iOS 和 Web 开发中很难见到，所以命令模式可能有点过时了。但是设计模式解决问题的过程，有助于为你自己的问题想出有创造力的解决方法。

### Patterns of Enterprise Application Architecture

*作者 Martin Fowler*

上一本书创造于桌面图形应用的时代，而这本书诞生于 Web 年代。书名看起来很枯燥，不过我发现此书包含了一系列有用的模式。读起来就像在实现一个类似于 Ruby on Rails 框架食谱，所以如果说 DHH 在写 Ruby on Rails 框架之前读过这本书，我也不会感到惊讶。

这些模式，已被用在 Web 的表单、HTML和数据库中。前两类很有趣，数据库模式还可以用于编写现代的 iOS 应用。如果你想了解一下类似于 Core Data（或者 ActiveRecord）的 ORM 是如何实现的，那可以看看这本书。比如，Core Data 使用了标识映射、延迟加载、元数据映射和查询对象等模式。像「四剑客」写的那本书一样，在写代码时虽然我们不会用到书中所有的模式，但是作者解决问题的这个过程，还是很令人兴奋的。

### Refactoring

*作者 Martin Fowler*

这本书也是由 Martin Fowler 编写的。它给重构下了一个准确的定义：

> 有人问我，“难道重构只是清理代码？”，在某种程度上，答案是肯定的，但我觉得重构更进了一步，因为它为清理代码提供了一个更加高效和更为可控的方式。

这本书还介绍了，重构是如何融入通常的软件开发过程中的：

> 使用重构来开发软件，需要把时间分为两个部分：添加功能和重构。添加的新功能时，不应该改变现有的代码；你只是添加新的功能。

在介绍和定义了重构之后，Fowler 深入讲解了一系列重构的例子。从抽取方法这种简单的开始，然后逐渐深入到类似引入空对象的重构方式。像之前的两本书一样，这本书从头读到尾也需要花些功夫。

### Domain-Driven Design

*作者 Eric Evans*

之前四本书籍大多介绍模式，这本书有一条小小的叙事线。一个开发者和一个领域的专家，搭建了一个管理船行程的应用。在这个过程中，从最初的研究阶段到实际的编码过程，你将学到如何把一个领域模型化。我从这本书了解到 value types 比 Swift 发布它的 value types 早了两年。

作者在程序员和领域专家之间编造的苏格拉底式对话也有助于我们理解。有人认为，在一个理想的世界中，一个产品经理可以在开发者和利益相关者之间传话。而在真实的世界中，你（作为开发者）对于表达软件的功能和局限也负有最终的责任，这本书展示类似的应该是什么样子。

### 如何思考 vs 思考什么

这五本书每一本都有各自的价值——你不仅能学到书里的知识，还能学会如何思考面临的问题。这些书都遵循着同样的结构：提出问题，然后给出解决方案。将问题和解决方法联系起来，能够看清如何解决的整个过程，最终学会如何处理其他问题。

> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。