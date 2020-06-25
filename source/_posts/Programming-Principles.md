---
title: 编程原则（Programming Principles）
date: 2018-10-04 14:26:25
tags: [Design Pattern]
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
* [童子军军规](#童子军军规)

### 模块间/类

* [最小化耦合](#最小化耦合)
* [迪米特法则](#迪米特法则)
* [组合优于继承](#组合优于继承)
* [正交性](#正交性)
* [稳健性原则](#稳健性原则)
* [控制反转](#控制反转)

### 模块/类

* [最大化聚合](#最大化聚合)
* [里氏代换原则](#里氏代换原则)
* [开放/封闭原则](#开放/封闭原则)
* [单一职责原则](#单一职责原则)
* [隐藏实现细节](#隐藏实现细节)
* [科里定律](#科里定律)
* [封装经常修改的代码](#封装经常修改的代码)
* [接口隔离原则](#接口隔离原则)
* [命令查询分离](#命令查询分离)

## KISS

大多数系统如果保持简单而不是复杂，效果最好。

为什么

* 更少的代码可以花更少的时间去写，Bug更少，并且更容易修改。
* 简单是复杂的最高境界。
* 完美境地，非冗杂，而不遗。

相关资料

* [KISS principle](http://en.wikipedia.org/wiki/KISS_principle)
* [Keep It Simple Stupid (KISS)](http://principles-wiki.net/principles:keep_it_simple_stupid)

## YAGNI

YAGNI的意思是“你不需要它”：在必要之前不要做多余的事情。

为什么

* 去做任何仅在未来需要的特性，意味着从当前迭代需要完成的功能中分出精力。
* 它使代码膨胀；软件变得更大和更复杂。

怎么做

* 在当你真正需要它们的时候，才实现它们，而不是在你预见到你需要它们的时候。

相关资料

* [You Arent Gonna Need It](http://c2.com/xp/YouArentGonnaNeedIt.html)
* [You’re NOT gonna need it!](http://www.xprogramming.com/Practices/PracNotNeed.html)
* [You ain't gonna need it](http://en.wikipedia.org/wiki/You_ain't_gonna_need_it)

## 做最简单的事情

为什么

* 仅有当我们只解决问题本身时，才能最大化地解决实际问题。

怎么做

* 扪心自问：“最简单的事情是什么？”。

相关资料

* [Do The Simplest Thing That Could Possibly Work](http://c2.com/xp/DoTheSimplestThingThatCouldPossiblyWork.html)

## 关注点分离

关注点分离是一种将计算机程序分离成不同部分的设计原则，以便每个部分专注于单个关注点。例如，应用程序的业务逻辑是一个关注点而用户界面是另一个关注点。更改用户界面不应要求更改业务逻辑，反之亦然。

引用[Edsger W. Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra) (1974)所说：

> 我有时将其称为“关注点分离”，即使这不可能完全做到，但它也是我所知道的唯一有效的思维整理技巧。这就是我所说的“将注意力集中在某个方面”的意思：这并不意味着忽略其他方面，只是对于从某一方面的视角公正地来看，另一方面是不相关的事情。

为什么

* 简化软件应用程序的开发与维护。
* 当关注点很好地分开时，各个部分可以被重用，并且可以独立开发和更新。

怎么做

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
* 使用[三法则（Rule of three）](http://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)).

相关资料

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

相关资料

* [Program optimization](http://en.wikipedia.org/wiki/Program_optimization)
* [Premature Optimization](http://c2.com/cgi/wiki?PrematureOptimization)

## 最小化耦合

模块/组件之间的耦合是它们互相依赖的程度，较低的耦合更好。换句话说，耦合是代码单元“B”在未知的代码单元“A”更改后“被破坏”的几率。

为什么

* 一个模块的更改通常会导致其他模块的更改，产生涟漪效益。
* 由于模块间的依赖性增加，模块装配可能需要更多的工作和/或时间。
* 特定的模块可能难以重用和/或测试，因为必须包含相关模块。
* 开发人员可能害怕更改代码，因为他们不确定什么会收到影响。

怎么做

* 消除，最小化和降低必要关联的复杂性。
* 通过隐藏实现细节，减少耦合。
* 使用[迪米特法则](#迪米特法则)。

相关资料

* [Coupling](http://en.wikipedia.org/wiki/Coupling_(computer_programming))
* [Coupling And Cohesion](http://c2.com/cgi/wiki?CouplingAndCohesion)

## 迪米特法则

不要和陌生人说话。

为什么

* 这通常会导致更紧密的耦合。
* 可能会暴露过多的实现细节。

怎么做

对象的方法只能调用以下方法：

1. 对象自身的方法。
2. 方法参数中的方法。
3. 方法中创建的任何对象的方法。
4. 对象的任何直接属性或字段的方法。

相关资料

* [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)
* [The Law of Demeter Is Not A Dot Counting Exercise](http://haacked.com/archive/2009/07/14/law-of-demeter-dot-counting.aspx/)

## 组合优于继承

为什么

* 类之间的耦合减少。
* 使用继承，子类很容易做出假设，并破坏里氏代换原则（LSP）。

怎么做

* 测试LSP（可替换性）以决定何时继承。
* 当存在“有”（或“使用”）的关系时使用组合，当存在“是”的关系时使用继承。

相关资料

* [Favor Composition Over Inheritance](http://blogs.msdn.com/b/thalesc/archive/2012/09/05/favor-composition-over-inheritance.aspx)

## 正交性

> 正交性的基本概念是，概念上不相关的东西在系统中不应该相关。

来源：[Be Orthogonal](http://www.artima.com/intv/dry3.html)

> 它越简单，设计越正交，异常就越少。这使得用编程语言学习、读写程序变得更容易。正交特征的含义是独立于环境；关键参数是对称性与一致性。

来源：<a herf="http://en.wikipedia.org/wiki/Orthogonality_(programming)">Orthogonality</a>

## 稳健性原则

> 坚持保守自己的作为，自由接受他人的作为。

合作的服务依赖于彼此的接口。通常，接口需要提升，导致另一端接收未指定的数据。如果接收到的数据没有严格遵守规范，那么简单的实现将仅拒绝合作。更复杂的实现却可以忽略它无法识别的数据。

为什么

* 为了能够提高服务，你需要确保提供者可以进行更改以支持新的需求，同时对现有客户端造成最小的破坏。

怎么做

* 向其他机器（或同一机器上的其他程序）发送指令或数据的代码应该完全符合规范，但接受输入的代码应接受不一致的输入，只要其意义明确。

相关资料

* [Robustness Principle in Wikipedia](https://en.wikipedia.org/wiki/Robustness_principle)
* [Tolerant Reader](http://martinfowler.com/bliki/TolerantReader.html)

## 控制反转

控制反转又被称为好莱坞原则，“不要打电话给我们，我们会打电话给你”。它是一种设计原则，计算机程序的自定义编写部分从通用框架接收控制流。控制反转具有强烈的含义，即可重用代码和特定于问题的代码是独立开发的，即使它们在应用程序中一同工作。

为什么

* 控制反转用于提高程序的模块性，使其具有可扩展性。
* 将任务的执行与实现分离。
* 将模块集中在其设计任务上。
* 使模块不受关于其他系统如何执行其任务的假设约束，而是依赖于约定。
* 以防止模块更换时出现副作用。

怎么做

* 使用工厂模式
* 使用服务定位器模式
* 使用依赖注入
* 使用依赖查找
* 使用模板方法模式
* 使用策略模式

相关资料

* [Inversion of Control in Wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)
* [Inversion of Control Containers and the Dependency Injection pattern](https://www.martinfowler.com/articles/injection.html)

## 最大化聚合

单个模块/组件的聚合性是其职责形成有意义的单元的程度，越高的聚合性越好。

为什么

* 增加了理解模块的难度。
* 增加了维护系统的难度，因为域中逻辑的更改会影响多个模块，并且一个模块的更改需要相关模块的更改。
* 由于大多数应用程序不需要模块提供的随机操作集，因此重用模块的难度增加。

怎么做

* 与组相关的功能共享一项职责（例如在一个类中）。

相关资料

* <a herf="http://en.wikipedia.org/wiki/Cohesion_(computer_science)">Cohesion</a>
* [Coupling And Cohesion](http://c2.com/cgi/wiki?CouplingAndCohesion)

## 里氏代换原则

里氏代换原则（LSP）完全是关于对象的预期行为：

> 程序中的对象应该可以替换为其子类型的实例，而不会改变该程序的正确性。

相关资源

* [Liskov substitution principle](http://en.wikipedia.org/wiki/Liskov_substitution_principle)
* [Liskov Substitution Principle](http://www.blackwasp.co.uk/lsp.aspx)

## 开放/封闭原则

软件实体（例如类）应对扩展是开放的，但对修改是封闭的。也就是说，这样的实体可以允许在不改变其源代码的情况下修改其行为。

为什么

* 通过最小化对现有代码的修改来提高可维护性和稳定性

怎么做

* 编写可以扩展的类（而不是可以修改的类）
* 只暴露需要更换的活动部分，隐藏其他所有部分。

相关资源

* [Open Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle)
* [The Open Closed Principle](https://8thlight.com/blog/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)

## 单一职责原则

一个类不应该有多个修改的原因。

长话版：每个类都应该有一个单独的职责，并且该职责应该完全由该类封装。职责可以定义为修改的原因，一次类或模块应该有且仅有一个修改的原因。

为什么

* 可维护性：仅有一个模块或类中需要修改。

怎么做

* 使用 [科里定律](#科里定律).

相关资料

* [Single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle)

## 隐藏实现细节

软件模块通过提供接口来隐藏信息（即实现细节），而不泄露任何不必要的信息。

为什么

* 当实现更改时，客户端使用的接口不必更改。

怎么做

* 最小化类和成员的可访问性。
* 不要公开成员数据。
* 避免将私有实现细节放入类的接口中。
* 减少耦合以隐藏更多实现细节。

相关资料

* [Information hiding](http://en.wikipedia.org/wiki/Information_hiding)

## 科里定律

科里定律是关于为任何特定代码选择一个明确定义的目标：仅做一件事。

* [Curly's Law: Do One Thing](http://blog.codinghorror.com/curlys-law-do-one-thing/)
* [The Rule of One or Curly’s Law](http://fortyplustwo.com/2008/09/06/the-rule-of-one-or-curlys-law/)

## 封装经常修改的代码

一个好的设计可以辨别出最有可能改变的热点，并将它们封装在API之后。当预期的修改发生时，修改会保持在局部。

为什么

* 在发生更改时，最小化所需的修改。

怎么做

* 封装API背后不同的概念。
* 将可能不同的概念分到各自的模块。

相关资料

* [Encapsulate the Concept that Varies](http://principles-wiki.net/principles:encapsulate_the_concept_that_varies)
* [Encapsulate What Varies](http://blogs.msdn.com/b/steverowe/archive/2007/12/26/encapsulate-what-varies.aspx)
* [Information Hiding](https://en.wikipedia.org/wiki/Information_hiding)

## 接口隔离原则

将臃肿的接口减少到多个更小更具体的客户端特定接口中。接口应该比实现它的代码更依赖于调用它的代码。

为什么

* 如果类实现了不需要的方法，则调用方需要了解该类的方法实现。例如，如果一个类实现了一个方法，但只是简单的抛出异常，那么调用方将需要知道实际上不应该调用这个方法。

怎么做

* 避免臃肿的接口。类不应该实现任何违反[单一职责原则](#单一职责原则)的方法。

相关资料

* [Interface segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)

## 童子军军规

美国童子军有一条简单的军规，我们可以使用到我们的职业中：“离开营地时比你到达时更干净”。根据童子军军规，我们应该至终保持代码比我们看到时更干净。

为什么

* 当对现有代码库进行更改时，代码质量往往会降低，从而积累技术债务。根据童子军军规，我们应该注意每一个提交（Commit）的质量。无论规模有多小，技术债务都会受到不断重构的抵制。

怎么做

* 每次提交都要确保它不会降低代码库的质量。
* 任何时候，如果有人看到一些代码不够清楚，他们就应该抓住机会在那里修复它。

相关资料
* [Opportunistic Refactoring](http://martinfowler.com/bliki/OpportunisticRefactoring.html)

## 命令查询分离

命令查询分离原则规定，每个方法都应该是执行操作的命令，或者是向调用者返回数据但不能同时做两件事的查询。提问不应该改变答案。  

利用这个原则，程序员可以更加自信地进行编码。查询方法可以在任何地方以任何顺序使用，因为它们不会改变状态。而使用命令，你必须更加小心。

为什么

* 通过将方法清晰地分为查询和命令，程序员可以在不了解每个方法的实现细节的情况下，更加自信地编码。

怎么做

* 将每个方法实现为查询或命令。
* 对方法名使用命名约定，该方法名表示该方法是查询还是命令。

相关资料

* [Command Query Separation in Wikipedia](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)
* [Command Query Separation by Martin Fowler](http://martinfowler.com/bliki/CommandQuerySeparation.html)
