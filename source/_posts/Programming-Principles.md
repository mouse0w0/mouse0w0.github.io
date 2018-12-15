---
title: 编程原则（Programming Principles）
date: 2018-10-04 14:26:25
tags: Programming
---

> **本文翻译自[Programming Principles](http://java-design-patterns.com/principles/)。**

每个程序员都可以从理解编程原理和模式中受益。这篇概述用于我个人参考，同时我也把它放在这。也许这在设计、讨论或复查中对你有所帮助。但请注意，这还远远不够，你常常需要在相互矛盾的原则之间做出权衡。

本文受[The Principles of Good Programming](http://www.artima.com/weblogs/viewpost.jsp?thread=331531)启发。我觉得这份列表已经足够了，但这并不完全符合我个人的想法。此外，我还需要更多的论证、细节以及其他资料的链接。如果您有任何反馈或者改进的建议，[请让我知道](https://github.com/webpro/programming-principles/issues)。

## 目录

### 通用

* [KISS (Keep It Simple Stupid)](#KISS)
* [YAGNI](#YAGNI)
* [做最简单的事情](#做最简单的事情)
* [关注点分离](#关注点分离)
* [保持事情不再重复](#保持事情不再重复)
* [为维护者写代码](#为维护者写代码)
* [避免过早优化](#避免过早优化)
* [Boy-Scout Rule](#boy-scout-rule)

### Inter-Module/Class

* [Minimise Coupling](#minimise-coupling)
* [Law of Demeter](#law-of-demeter)
* [Composition Over Inheritance](#composition-over-inheritance)
* [Orthogonality](#orthogonality)
* [Robustness Principle](#robustness-principle)
* [Inversion of Control](#inversion-of-control)

### Module/Class

* [Maximise Cohesion](#maximise-cohesion)
* [Liskov Substitution Principle](#liskov-substitution-principle)
* [Open/Closed Principle](#openclosed-principle)
* [Single Responsibility Principle](#single-responsibility-principle)
* [Hide Implementation Details](#hide-implementation-details)
* [Curly's Law](#curlys-law)
* [Encapsulate What Changes](#encapsulate-what-changes)
* [Interface Segregation Principle](#interface-segregation-principle)
* [Command Query Separation](#command-query-separation)

## KISS

大多数系统如果保持简单而不是复杂，效果最好。

为什么？

* 更少的代码可以花更少的时间去写，Bug更少，并且更容易修改。
* 简单是复杂的最高境界。
* 完美境地，非冗杂，而不遗。

> 译注：第二句话来自达·芬奇，原文为“Simplicity is the ultimate sophistication.”；第三句话来自安东尼·德·圣-埃克苏佩里，原文为“It seems that perfection is reached not when there is nothing left to add, but when there is nothing left to take away.”。

相关资料

* [KISS principle](http://en.wikipedia.org/wiki/KISS_principle)
* [Keep It Simple Stupid (KISS)](http://principles-wiki.net/principles:keep_it_simple_stupid)

## YAGNI

YAGNI的意思是“你不需要它”：在必要之前不要做多余的事情。

为什么？

* 去做任何仅在未来需要的特性，意味着从当前迭代需要完成的功能中分出精力。
* 它使代码膨胀；软件变得更大和更复杂。

怎么做？

* 在当你真正需要它们的时候，才实现它们，而不是在你预见到你需要它们的时候。

相关资料

* [You Arent Gonna Need It](http://c2.com/xp/YouArentGonnaNeedIt.html)
* [You’re NOT gonna need it!](http://www.xprogramming.com/Practices/PracNotNeed.html)
* [You ain't gonna need it](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it)

## 做最简单的事情

为什么？

* 仅有当我们只解决问题本身时，才能最大化地解决实际问题。

怎么做？

* 扪心自问：“最简单的事情是什么？”。

相关资料

* [Do The Simplest Thing That Could Possibly Work](http://c2.com/xp/DoTheSimplestThingThatCouldPossiblyWork.html)

## 关注点分离

关注点分离是一种将计算机程序分离成不同部分的设计原则，以便每个部分专注于单个关注点。例如，应用程序的业务逻辑是一个关注点而用户界面是另一个关注点。更改用户界面不应要求更改业务逻辑，反之亦然。

引用[Edsger W. Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra) (1974)所说：

> 我有时将其称为“关注点分离”，即使这完全不可能做到，但它也是我所知道的唯一有效的思维整理技巧。这就是我所说的“将注意力集中在某个方面”的意思：这并不意味着忽略其他方面，只是对于从某一方面的视角公正地来看，另一方面是不相关的事情。

为什么？

* 简化软件应用程序的开发与维护。
* 当关注点很好地分开时，各个部分可以被重用，并且可以独立开发和更新。

怎么做？

* 将程序功能分成联系部分尽可能少的模块。

相关资料

* [Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)

## 保持事情不再重复

在一个系统内，每一项认识都必须有一个单一的、明确的、权威的表示。

程序中的每一项重要功能都应该只在源代码中的一个地方实现。相似的函数由不同的代码块执行的情况下，抽象出不同的部分，将它们组合为一个函数通常是有益的。

为什么

* 重复（无意或有意的重复）会造成噩梦般的维护，保养不良和逻辑矛盾。
* 对系统中任意单个元素的修改不需要改变其他逻辑上无关的元素。
* 此外，相关逻辑的元素的变化都是可预测的和均匀的，因此是保持同步的。

怎么做

* 只在一个处编写业务规则、长表达式、if语句、数学公式、元数据等。
* 确定系统中使用的每一项认识的唯一来源，然后使用该源来生成该认识的适用实例（代码、文档、测试等）。
* Identify the single, definitive source of every piece of knowledge used in your system, and then use that source to generate applicable instances of that knowledge (code, documentation, tests, etc).
* 使用[三法则（Rule of three）](http://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)).

相关资源

* [Dont Repeat Yourself](http://c2.com/cgi/wiki?DontRepeatYourself)
* [Don't repeat yourself](http://en.wikipedia.org/wiki/Don't_repeat_yourself)
* [Don't Repeat Yourself](http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself)

相似资料

* [Abstraction principle](http://en.wikipedia.org/wiki/Abstraction_principle_(computer_programming))
* [Once And Only Once](http://c2.com/cgi/wiki?OnceAndOnlyOnce) is a subset of DRY (also referred to as the goal of refactoring).
* [Single Source of Truth](http://en.wikipedia.org/wiki/Single_Source_of_Truth)
* A violation of DRY is [WET](http://thedailywtf.com/articles/The-WET-Cart) (Write Everything Twice)

## 为维护者写代码

为什么

* 到目前为止，维护是任何项目中最昂贵的阶段。

怎么做

* *成为*维护者。
* 不论何时编写代码，要想着最后维护代码的人是一个知道自己住在哪里的暴力精神病人。
* 如果某个入门的人掌握了代码，他们就会从阅读和学习代码中获得乐趣，以这样的想法去编写代码和注释。
* [别让我想（Don't make me think）](http://www.sensible.com/dmmt.html).
* 使用[最少惊讶原则（Principle of Least Astonishment）](http://en.wikipedia.org/wiki/Principle_of_least_astonishment).

相关资料

* [Code For The Maintainer](http://c2.com/cgi/wiki?CodeForTheMaintainer)
* [The Noble Art of Maintenance Programming](http://blog.codinghorror.com/the-noble-art-of-maintenance-programming/)

## 避免过早优化

引用[Donald Knuth](http://en.wikiquote.org/wiki/Donald_Knuth)所说：

> 程序员浪费大量的时间来思考或担心程序的非关键部分的速度，而考研尝试这些优化实际上在调试和维护时有很强的负面影响。比如说在97%的开发时间，我们应该忽略低效率：过早的优化是万恶之源。然而，我们不应该在关键的3%中放弃我们的机会。

当然，需要理解什么是“过早”什么不是“过早”。

为什么

* 瓶颈在哪是未知的。
* 优化后，阅读和维护可能会更困难。

怎么做

* [使它运作，使它正确，使它更快（Make It Work Make It Right Make It Fast）](http://c2.com/cgi/wiki?MakeItWorkMakeItRightMakeItFast)
* 不要在你不需要的时候优化，只有在你发现一个瓶颈之后才能优化它。

相关资源

* [Program optimization](http://en.wikipedia.org/wiki/Program_optimization)
* [Premature Optimization](http://c2.com/cgi/wiki?PrematureOptimization)

## Minimise Coupling

Coupling between modules/components is their degree of mutual interdependence; lower coupling is better. In other words, coupling is the probability that code unit "B" will "break" after an unknown change to code unit "A".

Why

* A change in one module usually forces a ripple effect of changes in other modules.
* Assembly of modules might require more effort and/or time due to the increased inter-module dependency.
* A particular module might be harder to reuse and/or test because dependent modules must be included.
* Developers might be afraid to change code because they aren't sure what might be affected.

How

* Eliminate, minimise, and reduce complexity of necessary relationships.
* By hiding implementation details, coupling is reduced.
* Apply the [Law of Demeter](#law-of-demeter).

Resources

* [Coupling](http://en.wikipedia.org/wiki/Coupling_(computer_programming))
* [Coupling And Cohesion](http://c2.com/cgi/wiki?CouplingAndCohesion)

## Law of Demeter

Don't talk to strangers.

Why

* It usually tightens coupling
* It might reveal too much implementation details

How

A method of an object may only call methods of:

  1. The object itself.
  1. An argument of the method.
  1. Any object created within the method.
  1. Any direct properties/fields of the object.

Resources

* [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)
* [The Law of Demeter Is Not A Dot Counting Exercise](http://haacked.com/archive/2009/07/14/law-of-demeter-dot-counting.aspx/)

## Composition Over Inheritance

Why

* Less coupling between classes.
* Using inheritance, subclasses easily make assumptions, and break LSP.

How

* Test for LSP (substitutability) to decide when to inherit.
* Compose when there is a "has a" (or "uses a") relationship, inherit when "is a".

Resources

* [Favor Composition Over Inheritance](http://blogs.msdn.com/b/thalesc/archive/2012/09/05/favor-composition-over-inheritance.aspx)

## Orthogonality

> The basic idea of orthogonality is that things that are not related conceptually should not be related in the system.

Source: [Be Orthogonal](http://www.artima.com/intv/dry3.html)

> It is associated with simplicity; the more orthogonal the design, the fewer exceptions. This makes it easier to learn, read and write programs in a programming language. The meaning of an orthogonal feature is independent of context; the key parameters are symmetry and consistency.

Source: [Orthogonality](http://en.wikipedia.org/wiki/Orthogonality_(programming))

## Robustness Principle

> Be conservative in what you do, be liberal in what you accept from others

Collaborating services depend on each others interfaces. Often the interfaces need to evolve causing the other end to receive unspecified data. A naive implementation refuses to collaborate if the received data does not strictly follow the specification. A more sophisticated implementation will still work ignoring the data it does not recognize.

Why

* In order to be able to evolve services you need to ensure that a provider can make changes to support new demands while causing minimal breakage to their existing clients.

How

* Code that sends commands or data to other machines (or to other programs on the same machine) should conform completely to the specifications, but code that receives input should accept non-conformant input as long as the meaning is clear.

Resources

* [Robustness Principle in Wikipedia](https://en.wikipedia.org/wiki/Robustness_principle)
* [Tolerant Reader](http://martinfowler.com/bliki/TolerantReader.html)

## Inversion of Control

Inversion of Control is also known as the Hollywood Principle, "Don't call us, we'll call you". It is a design principle in which custom-written portions of a computer program receive the flow of control from a generic framework. Inversion of control carries the strong connotation that the reusable code and the problem-specific code are developed independently even though they operate together in an application. 

Why

* Inversion of control is used to increase modularity of the program and make it extensible.
* To decouple the execution of a task from implementation.
* To focus a module on the task it is designed for.
* To free modules from assumptions about how other systems do what they do and instead rely on contracts.
* To prevent side effects when replacing a module.

How

* Using Factory pattern
* Using Service Locator pattern
* Using Dependency Injection
* Using contextualized lookup
* Using Template Method pattern
* Using Strategy pattern

Resources

* [Inversion of Control in Wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)
* [Inversion of Control Containers and the Dependency Injection pattern](https://www.martinfowler.com/articles/injection.html)

## Maximise Cohesion

Cohesion of a single module/component is the degree to which its responsibilities form a meaningful unit; higher cohesion is better.

Why

* Increased difficulty in understanding modules.
* Increased difficulty in maintaining a system, because logical changes in the domain affect multiple modules, and because changes in one module require changes in related modules.
* Increased difficulty in reusing a module because most applications won’t need the random set of operations provided by a module.

How

* Group related functionalities sharing a single responsibility (e.g. in a class).

Resources

* [Cohesion](http://en.wikipedia.org/wiki/Cohesion_(computer_science))
* [Coupling And Cohesion](http://c2.com/cgi/wiki?CouplingAndCohesion)

## Liskov Substitution Principle

The LSP is all about expected behavior of objects:

> Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

Resources

* [Liskov substitution principle](http://en.wikipedia.org/wiki/Liskov_substitution_principle)
* [Liskov Substitution Principle](http://www.blackwasp.co.uk/lsp.aspx)

## Open/Closed Principle

Software entities (e.g. classes) should be open for extension, but closed for modification. I.e. such an entity can allow its behavior to be modified without altering its source code.

Why

* Improve maintainability and stability by minimizing changes to existing code.

How

* Write classes that can be extended (as opposed to classes that can be modified).
* Expose only the moving parts that need to change, hide everything else.

Resources

* [Open Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle)
* [The Open Closed Principle](https://8thlight.com/blog/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)

## Single Responsibility Principle

A class should never have more than one reason to change.

Long version: Every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class. Responsibility can be defined as a reason to change, so a class or module should have one, and only one, reason to change.

Why

* Maintainability: changes should be necessary only in one module or class.

How

* Apply [Curly's Law](#Curly-s-Law).

Resources

* [Single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle)

## Hide Implementation Details

A software module hides information (i.e. implementation details) by providing an interface, and not leak any unnecessary information.

Why

* When the implementation changes, the interface clients are using does not have to change.

How

* Minimize accessibility of classes and members.
* Don’t expose member data in public.
* Avoid putting private implementation details into a class’s interface.
* Decrease coupling to hide more implementation details.

Resources

* [Information hiding](http://en.wikipedia.org/wiki/Information_hiding)

## Curly's Law

Curly's Law is about choosing a single, clearly defined goal for any particular bit of code: Do One Thing.

* [Curly's Law: Do One Thing](http://blog.codinghorror.com/curlys-law-do-one-thing/)
* [The Rule of One or Curly’s Law](http://fortyplustwo.com/2008/09/06/the-rule-of-one-or-curlys-law/)

## Encapsulate What Changes

A good design identifies the hotspots that are most likely to change and encapsulates them behind an API. When an anticipated change then occurs, the modifications are kept local.

Why

* To minimize required modifications when a change occurs

How

* Encapsulate the concept that varies behind an API
* Possibly separate the varying concept into its own module

Resources

* [Encapsulate the Concept that Varies](http://principles-wiki.net/principles:encapsulate_the_concept_that_varies)
* [Encapsulate What Varies](http://blogs.msdn.com/b/steverowe/archive/2007/12/26/encapsulate-what-varies.aspx)
* [Information Hiding](https://en.wikipedia.org/wiki/Information_hiding)

## Interface Segregation Principle

Reduce fat interfaces into multiple smaller and more specific client specific interfaces. An interface should be more dependent on the code that calls it than the code that implements it.

Why

* If a class implements methods that are not needed the caller needs to know about the method implementation of that class. For example if a class implements a method but simply throws then the caller will need to know that this method shouldn't actually be called.

How

* Avoid fat interfaces. Classes should never have to implement methods that violate the [Single responsibility principle](#single-responsibility-principle).

Resources

* [Interface segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)

## Boy-Scout Rule

The Boy Scouts of America have a simple rule that we can apply to our profession: "Leave the campground cleaner than you found it". The boy-scout rule states that we should always leave the code cleaner than we found it.

Why

* When making changes to an existing codebase the code quality tends to degrade, accumulating technical debt. Following the boyscout rule, we should mind the quality with each commit. Technical debt is resisted by continuous refactoring, no matter how small.

How

* With each commit make sure it does not degrade the codebase quality.
* Any time someone sees some code that isn't as clear as it should be, they should take the opportunity to fix it right there and then.

Resources

* [Opportunistic Refactoring](http://martinfowler.com/bliki/OpportunisticRefactoring.html)

## Command Query Separation

The Command Query Separation principle states that each method should be either a command that performs an action or a query that returns data to the caller but not both. Asking a question should not modify the answer.

With this principle applied the programmer can code with much more confidence. The query methods can be used anywhere and in any order since they do not mutate the state. With commands one has to be more careful.

Why

* By clearly separating methods into queries and commands the programmer can code with additional confidence without knowing each method's implementation details.

How

* Implement each method as either a query or a command
* Apply naming convention to method names that implies whether the method is a query or a command

Resources

* [Command Query Separation in Wikipedia](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)
* [Command Query Separation by Martin Fowler](http://martinfowler.com/bliki/CommandQuerySeparation.html)
