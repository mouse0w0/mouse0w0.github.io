---
title: 介绍Mixin——混淆与Mixin
date: 2018-11-02 12:39:35
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Introduction to Mixins Obfuscation and Mixins](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Obfuscation-and-Mixins)

在我们进一步研究Mixin架构之前，先让我们绕过它去讨论另一个重要主题：Minecraft代码库中的*混淆*以及它与Mixin的关系。

> *混淆（Obfuscation）*是将原本人类可读的代码符号转换为混淆的代码符号的过程，使得人难以阅读它（事实上，*混淆*这个词的意思仅仅是“*故意使模糊*”）。

### 1. 在混淆时代的开发生命周期

由于Minecraft是使用Java编写的，如果不使用混淆技术，则很容易将其反编译为可读代码。Mojang在发布之前对Minecraft进行了混淆，这对模组开发者造成了问题，原因有两个：

1. 混淆的使用将所有的内容都放在“默认包”中，这使得无法从代码库`import`类。
2. 使用混淆后的名称将是一场噩梦，因为代码基本上不可读。

这就意味着，为了能够基于Minecraft编译我们的代码，有必要预先对Java类进行**反**混淆处理，一个名为*Mod Coder Pack*（简称MCP）的社区项目提供了遍历的方法。

一旦我们编写了代码，我们需要**重**混淆我们的Mod代码，以便它可以与原始的（混淆的）代码库一起工作。因此，开发生命周期看起来如下图所示：

![混淆生命周期](./Introduction-to-Mixins-Obfuscation-and-Mixins/obfuscation_lifecycle.png)

### 2. 化解识别危机 - 定义“混淆边界”

让我们从基础知识开始：

当在开发Minecraft Mod时，字段和方法**至多有3个名称**：

1. **混淆名（简称“OBF名”）** - 这是Mojang分配的名称，作为其*混淆*代码库的一部分，它通常只有1-2个字母长，例如，一个混淆方法也许名为`k`。

2. **“Searge名”（简称“SRG名”）** - 这是一个分配给字段或方法的唯一的代号，以便使反编译成为可能。它由前缀、唯一ID和成员的（原始）名称组成，例如`func_1234_k`。

3. **“MCP名”** - 这是一个来自社区的更可读的名字，以便使代码库更容易理解。例如`getHealth`。

在反编译过程中，字段将从一种名称转换为另一种名称，最终止于“友好的”*MCP名*。在重混淆过程中，反向进行处理。在我们的示例中，方法`k`变为`func_1234_k`最终变为`getName`。

在每个阶段**所有**字段和方法被重命名，因此每组混淆形成不相关的混淆*环境*，所有字段和方法具有对应于该环境的名称：
  
![混淆环境](./Introduction-to-Mixins-Obfuscation-and-Mixins/obfuscation_environments.png)

我们还将把这些假想环境之间的假想边界称为“*混淆边界（Obfuscation Boundary）* ”，因为显然*跨越*边界可能会造成问题。例如，方法`getHealth`（MCP名）总是希望`takeDamage`方法在任何特定执行周期都是它的MCP名称，如果同时存在来自不同环境的名称，那么很有可能发生问题。

### 3. 如何工作

混淆环境之间的转换必须“同时进行”，并且有*映射文件（Mapping File）*的协助，文件中包含一个名称到另一个名称的映射。这些映射文件包含一个条目，用于代码库中的*每一个*字段、方法、参数和类。

> 严格来说，这并不正确，因为MCP名是多人管理的，所以有很多成员没有定义MCP名，但是现在我们假装所有混淆都一直存在，因为现在并不需要讨论该例外。让我们设想一下，未映射的MCP名与SRG名是相同的。

为了确保不仅重命名了符号的声明，而且重命名了对该符号的所有引用，必须同时将重映射应用于整个代码库。虽然SRG名是唯一的并且可以被确定性地重映射，但其他的符号并不那么简单，因此重映射工具需要同时载入和理解整个代码库以便有效进行重映射。

正因为工具以这种方式工作，对代码结构和关系*有*一个基本的了解（例如，派生类中重写的方法），所以重映射器才能够重映射对混淆类的引用，甚至在不属于源代码库的类中。通过这种方式，派生类（例如我们在模组中添加的类）和有对重映射方法调用的类（比如我们可以在模组中添加的类）*也*将重映射这些方法的调用和字段的访问！

### 4. 与Mixin的关系

你可能想知道为什么你需要了解上述内容，它和Mixin有什么关系？现在你应该清楚的是：

1. 要以某种方式与游戏类交互的**所有内容**都**必须**通过混淆工具才能用于生产环境。
2. 任何**直接引用**游戏中的字段、方法和类的东西，都将由重映射器自动处理，因为重映射器已经**理解**了这些关系。

然而，这并不符合Mixin的情况，因为我们可以在Mixin中创建影子字段和方法，而不是直接引用它们的对应物！

#### 4.1 处理影子字段和方法的混淆

你可能会回想起本系列[第一部分](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture#5-to-light-a-candle-is-to-cast-a-shadow)，可以将*影子*成员添加到我们的Mixin中，以指定在运行时目标类中存在的特定方法或字段。这导致的主要问题是混淆器没有建立对这些成员的理解，从而将无法自动混淆它们。

Mixin通过在编译时解析`@Shadow`注解，并将影子成员的适当混淆条目直接添加到混淆表中，以解决这个问题。这是由[注解处理器](https://github.com/SpongePowered/Mixin/wiki/Using-the-Mixin-Annotation-Processor)处理的，它插入到Java编译器中。

众所周知，混淆器已经能够理解派生类中的字段和方法的引用，因此我们只需要为影子成员本身添加混淆表条目，然后自动处理对那些成员的引用。然后，Mixin可以安全地通过混淆边界。

#### 4.2 其他Mixin特性的混淆

在本系列之后的内容，你将了解到其他Mixin特性，这些特性需要通过特殊处理来通过混淆边界。在此之上要记住的关键是：

* 你的Mixin中的任何**直接引用**游戏代码库中的类将被自动处理，例如：
 
 * 当Mixin从游戏类派生时，引用父类方法。
 * 你的Mixin中任何复写了游戏类或接口中的方法的`@Override`方法。
 * 你的Mixin中任何对游戏类或成员的外部引用。

* 任何**Mixin特定机制**，例如*Shadow*、*Overwrite*（将在下一节中介绍）和*Injector*（将在稍后介绍），将始终使用某种注解进行修饰。这使得[Mixin注解处理器（Annotation Processor）](https://github.com/SpongePowered/Mixin/wiki/Using-the-Mixin-Annotation-Processor) 能够发现它们，它将处理它们的混淆。

### 5. The Nitty Gritty

如果你正在阅读本系列的介绍，你应该在这里停一下。以下部分提供了一些更详细的技术细节，并且仅出于完整性考虑，这将在之后的部分中引用。它们并不打算仅仅只是介绍一下而已，现在已经给你打过预防针了。

#### 5.1 “硬”与“软”混淆引用

传递到SpecialSource[<sup>1</sup>](#note1)的符号引用当然会像我们预期的那样被重混淆，并且底层字节码中的引用也会因此被重混淆。这种“硬”的重混淆适用于以下类型的成员：

* 类引用（仅当混淆到“Notch名”[<sup>2</sup>](#note2)时，不会应用于Forge）
* 方法名
* 字段名

但是，某些成员引用在注解中定义为字符串，特别是：

* Injector声明
* Rerouter声明

由于SpecialSource不能重映射这些“软”引用，所以使用不同的机制

##### 5.1.1 Mixin引用映射

为了允许混淆“软”引用，[注解处理器](https://github.com/SpongePowered/Mixin/wiki/Using-the-Mixin-Annotation-Processor)处理映射文件，该映射文件被包含在生产用Jar中，并在[配置文件](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment#mixin-configuration-files)中指定。这个*引用表（Reference Map，或简称“refmap”）*包含Mixin集合中所有到它们混淆的对象的软引用。

每个编译阶段都放出一个引用映射，因此在对应转换期间编译的每个Mixin集合都应该对该转换使用相同的引用映射。应该选择引用映射的唯一名称以避免冲突。

例如，假设我们在模组中定义如下的Mixin集合：

    mixins.myproject.core.json
    mixins.myproject.extra.json

我们可以为这两个集合定义一个引用映射文件，并将其命名为`mixins.myproject.refmap.json`以使其一致。

注意，在生产用Jar中包含引用映射文件是绝对重要的，并要在Mixin配置中制定他。如果不这样做，将会在Mixin应用时导致错误，因为Mixin的引用将不由Mixin处理器解决。

在如下情况下，可以省略引用映射文件：

* 你的Mixin里没有使用Injector和Rerouter。

#### 5.2  运行时反混淆与Mixin

一些特定环境使用部分运行时反混淆。也就是说，它们在运行时将符号反混淆为中间名（SRG名），而其他的则不反混淆。这个部分的转换是为了让模组能够有更稳定的混淆环境，以实现跨多个版本的游戏的目标。

显然，在应用运行时反混淆后，将Mixin字节码和目标类组合是非常重要的，因此环境中的混淆映射需与Mixin中的混淆映射相匹配。让我们重新看看前文中的图，转换器链中的Mixin概述：

![](./Introduction-to-Mixins-Obfuscation-and-Mixins/mixin_transformer_chain.png)

当我们考虑在此图中（在上游转换器链中）在哪里应用运行时反混淆时，我们可以看到反混淆转换器本身如何表示混淆边界，以及为什么必须在该转换器的下游应用Mixin：

![](./Introduction-to-Mixins-Obfuscation-and-Mixins/mixin_transformer_chain_obf.png)

#### 5.3  不可预测的成员名称 - 合成把戏的困扰

OBF -> SRG -> MCP的确定性规则的例外是目标类中的合成成员（Synthetic Member）[<sup>3</sup>](#note3)造成的。虽然混淆代码库中的合成成员也像它们的外部类一样具有混淆名称，但是在开发中会出现问题，因为重新建立的内部类关系导致这些合成成员被剥离，然后被编译器重新创建。

例如，让我们思考一个非静态内部类对其外部类的引用，通常在由`javac`生成的类中将其命名为`this$0`。当被混淆时，这个成员会得到一个使人混乱的名称“`a`”，并且在反编译过程中交替地重命名为“`field_999_a`”，最后才得到易于理解的MCP名称“`myOuter`”。然而，由于在建立开发工作环境的最后阶段就是在源代码中重新集成内部类及其外部类，因此，最终将剥离合成字段，并允许编译器重新创建该字段，从而在原先的“`this$0`”开发工作空间中给该字段一个名称。

这造成了一个问题，如果我们希望影射该字段，那么必须将其命名为`myOuter`（因为映射文件中出现的值就是这样命名的），但如果这样做，那么Shadow在开发时就不能工作，因为实际上并不存在名为`myOuter`的字段！

##### 5.3.1 别名

可以通过指定影子字段的*别名*来解决这个问题。当试图定位影子字段的目标时，别名作为Mixin处理器的最后一个解决方案存在。如果Mixin处理器无法在目标类中找到所需的字段，那么它会首先检查别名列表，然后再出现错误。

若要指定Shadow或者Overwrite注解的别名，只需要在注释上确定`aliases`的值：
```java
@Shadow(aliases = {"this$0"})
private MyOuterClassType myOuter;
```

请注意，别名只能用于`private`字段和方法。这是因为别名只能在Mixin应用时解析，因此字段的重命名只能涉及到所包含的类，而不能进一步扩散（因为此时可能已经加载并使用了派生的类Mixin或其他引用类）。然而，这通常不是问题，因为作为别名机制的起因的合成字段几乎总是私有的或者是包私有的。

> 译者注：
> 
> <a name="note1"><sup>1</sup></a>SpecialSource是Jar混淆映射的自动生成器和重命名器，通常被用于Minecraft反混淆和重混淆的混淆映射表的生成。
>
> <a name="note2"><sup>2</sup></a>Notch名其实就是混淆名（又叫OBF名）。
>
> <a name="note3"><sup>3</sup></a>合成成员（Synthetic Member）是由Java编译器生成的一种特殊的字段（如非静态内部类就有一个引用父类的合成字段）、方法（例如泛型的桥方法，且内部类引用了外部类的字段或方法都会产生相应的合成方法）或类。
