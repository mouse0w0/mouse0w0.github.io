---
title: Mixin介绍——Mixin环境
date: 2018-11-01 17:18:21
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Introduction to Mixins The Mixin Environment](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment)

现在我们已经了解了Mixin的[基本特性](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture/)（提示：如果你还没有读过，请返回并先阅读它！），让我们快点绕开它，来了解如何使用我们的新创建的Mixin进入游戏环境。

#### Mixin是如何工作的

正如我在上一篇[文章](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture/)中所提到的，通常情况下，最好记住**Mixin不是严格意义上的类**，也不是在运行时加载的类，而是使用[ASM](http://asm.ow2.org/)的Tree API解析的原始字节码。这个API生成一个基于节点的底层字节码视图，然后可以通过Mixin转换器（Transformer）[<sup>1</sup>](#note1)合并到目标类中。

Mixin本身是独立于常规类，通过转换器链（Transformer Chain）[<sup>2</sup>](#note2)传输的，这是为了允许重映射转换器（Remapping Transformer）[<sup>3</sup>](#note3)在Mixin与目标类组合之前完成它们的工作。因此，Mixin转换器在任何重映射转换器的*下游*是很重要的，这是通过在转换器链中插入代理转换器（Proxy Transformer）来内部处理的。

![Mixin转换器链](mixin_transformer_chain.png)

Mixin需要在类开始载入之前进行处理，以便识别它们的**目标类**，进行[**层次结构验证**](https://github.com/SpongePowered/Mixin/wiki/About-Hierarchy-Validation-in-Mixins)，并为方法和字段解析任何需要更新的**静态绑定**。你希望使用的Mixin应该在配置文件中指定，你的CoreMod、Tweaker、Jar Metadata或litemod.json将指定模组Jar内的资源位置。

> 配置路径被作为*资源*加载，因此可以放在Jar任何包中。重要的是，资源路径必须是相对的，**不要**在文件路径上预先准备一个主斜杠。
>
> 例如，如果您的配置文件在包`com.somepackage`中，则应该将配置文件的路径指定为`com/somepackage/myconfig.json`。

每个**配置**定义一个**Mixin集合**，并且对于你的应用程序，可能会有多个**Mixin集合**。当你的Mixin针对不同的*环境*（见下文）时，只*需要*分离Mixin到不同集合中。然而有时也为了组织结构而将Mixin拆分到不同集合。

### Mixin配置文件

除了定义**Mixin集合**之外，Mixin配置文件还定义了集合的附加属性。

首先，用四个键来定义集合本身：

* `package`为这组Mixin定义了父包（这很重要，因为该包和所有子包在运行时都将被排除在`LaunchClassLoader`之外）。
* `mixins`为该配置定义了位于*父包*内的*Mixin“类”列表*，每个Mixin“类”都相对于*父包*指定，可以使用子包。此列表中的每一个条目都将应用于客户端和专用服务端（Dedicated Server）[<sup>4</sup>](#note4)侧。
* `client`定义了一组**仅**应用于**客户端**侧的Mixin。
* `server`定义了一组**仅**应用于**专用服务端**侧的Mixin。

然后可以定义附加属性：

* `refmap`定义该集合的**引用映射（Reference Map）**文件名（关于如何使用Mixin处理混淆之后再做更多讨论）。
* `priority`定义了这个Mixin集合相对于其他配置的**优先级（Priority）**。
* `plugin`是可选的用于Mixin配置的**伴随插件**的类名，该类可以在运行时以编程方式调整Mixin配置，参见~~Mixin伴随插件~~（尚未完成）。
* `required`定义了Mixin集合是否**必须**。当*required*标志应设为`true`时，如果单个Mixin未能应用，则应视为整个游戏处于失败状态。
* `minVersion`在当Mixin集合使用一些在特定版本添加的Mixin功能时设置。它可以省略当前版本不可用的Mixin集合。
* `setSourceFile`使Mixin处理器可以用Mixin类的源代码文件覆盖目标类中的`source file`属性。这在调试Mixin时很有用。
* `verbos`将所有`DEBUG`级别的日志消息等级调整为Mixin集合的`INFO`级。这也可以通过`mixin.debug.verbose`[系统属性](https://github.com/SpongePowered/Mixin/wiki/Mixin-Java-System-Properties)在全局范围内启用。

需要注意的最重要的一点是，Mixin“类”必须位于一个包中，且其中没有你的项目的其他类（这包括Mixin主包的子包），因为启动时整个包将被排除在转换类加载器（Transforming Classloader）之外。

### 游戏生命周期与你

为了理解Mixin如何与游戏底层交互，首先理解运行在`LaunchWrapper`中的游戏生命周期是至关重要的。

*LaunchWrapper[<sup>5</sup>](#note5)*接管了游戏的正常启动过程，以便允许*Tweaker*[<sup>6</sup>](#note6)修改游戏。每个*Tweaker*可以提供*类转换器（Class Transformer）[<sup>1</sup>](#note1)*，这能够在游戏类加载时修改它。*Tweaker*在命令行中指定，并由`Launch`类逐个加载并初始化，该类构成了*LaunchWrapper*启动逻辑的核心。

像FML和LiteLoader这样的*Tweaker*还能够加载和注入它们找到的其他*Tweaker*，因此`Launch`在循环中初始化*Tweaker*，直到不再继续注册更多的*Tweaker*。我们可以想到`Launch`中的执行流程如下：

![启动生命周期](mixin_env_0.png)

这个初始化阶段*必须*在游戏开始加载之前完成，这是绝对必要的，否则因为在*Tweak*初始化周期中晚点注册的*Transformer*可能没机会处理它需要的类，并且转换器链也可能不完整。

在初始化阶段发生的事情：

* 像FML和LiteLoader这样的Tweaker寻找Mod，并且注入它们的转换器以及任何它们找到的额外的Tweaker。
* FML的“Core Mod”（又叫载入时插件（Loading Plugin））被初始化并可以注册它们自己的转换器。

由于Mixin子系统必须在这个早期阶段（在游戏类加载之前）初始化，所以它必须作为CoreMod或Tweaker加载，具体参阅下面的引导部分。

![启动生命周期](mixin_env_1.png)

一旦*Tweaker*初始化完成，`Launch`将调用游戏的原始`main()`方法，该方法启动游戏加载过程。在此之前，转换器链已经完成，因此转换器链可以通过注册的转换器加载和处理游戏类。游戏进入其主循环，按需加载类，并由转换器链（包括Mixin）处理。

上述内容中应该明白的是，游戏运行因此被分成两个截然不同的*阶段*，*Tweaker*在**预初始化阶段（pre-init phase）**被初始化，以及类似于正常的游戏生命周期的**默认阶段（default phase）**，该阶段可不在LaunchWrapper中运行。有必要了解这些阶段及其与Mixin子系统的关系。

#### 这只是我经历的一个阶段

在通常情况下，我们的Mixin设置将只处理在**默认**阶段加载的类，并且事情非常简单。在这个阶段，我们已知：

* 转换器链是完整的
* 游戏类能被安全地载入

这意味着加载和转换游戏类字节码并应用Mixin是安全的，加载那些以便应用Mixin的必须的*类元数据*也是安全的（请记住，从上文开始，Mixin的验证和预转换在启动时一次性完成）。如果我们过早地生成元数据（例如，在预初始化之前的阶段），那么就有可能没有注册关键的转换器！这将导致Mixin字节码不完整或无效。

_那么为什么要考虑**预初始化**呢？_

答案很简单：*“那么我们可以融入到其他Tweaker提供的核心类中，特别是FML”*

像*Sponge*这样的平台的要求之一是，有时必须以某种方式挂钩到底层平台，如果转换发生在**Default**阶段，核心类就已经加载（这已经超出了转换器的范围）。然而，正如我们知道的，在**预初始化**阶段加载与游戏相关的Mixin将不会起作用，因为需要的转换器不存在。

#### 关于环境

然后，在Mixin处理器中，我们确实将处理过程分割为不同_**环境**_（每个阶段一个），然后你可以针对所期望应用的*环境*拆分你的Mixin集合。典型的Mixin应用情景的解决方式如下：

* 一个只组合游戏类的Mixin集合应该在**默认**环境中注册
* 当需要在两个阶段应用Mixin时，两个Mixin集合（配置）应该将一个集合指定为**预初始化**Mixin，另一个集合则指定为**默认**Mixin。

#### 跃跃欲试

值得注意的是，FML作为一个*Tweaker*加载它的CoreMod，并使用代理*Tweaker*将它们重新注入到启动生命周期中。因为我们对启动顺序可能有点微妙的感觉，因此值得注意的是，*“第一类（first-class）”*Tweaker[<sup>7</sup>](#note7)和CoreMod初始化期间产生的间接Tweaker有细微区别。

Mixin库内含一个第一类的Tweaker，它被设计为在尽可能早的时候注册并开始处理类，这在下图中标记为 **(1)** 。正常的CoreMod启动发生于 **(2)** 处的第一次循环，然后游戏加载开始于 **(3)**。

![启动生命周期](mixin_env_2.png)

如果需要在**预初始化**与类组合，则建议使用第一类Tweaker。

### 使用FML CoreMod引导Mixin

> **注意：在生产环境中通过CoreMod引导Mixin目前是不可能的，要在发布版本中使用Mixin和Forge，你必须使用下述的调整方法。**但是，你可能希望通过CoreMod在开发工作空间中加载Mixin，这可能实现，但它必须在生产环境中使用Tweaker：

使用CoreMod在开发环境中引导Mixin：

* 在你的CoreMod的构造函数中，使用以下代码初始化Mixin环境并设置Mixin配置文件的名称：
```java
public MyCoreMod() {
    // 该语句必须最先出现，不含有该语句
    // 将导致一个运行时错误
    MixinBootstrap.init();
    
    // 检索默认的Mixin环境和注册配置文件
    MixinEnvironment.getDefaultEnvironment()
        .addConfiguration("mixins.mymod.json");
}
```
 
### 通过第一类Tweaker引导Mixin

Mixin库内建有第一类*Tweaker*，你可以通过在Mod的Jar元数据中指定Tweaker类来使用。为此，只需像如下所示简单地指定Tweaker类名和你的Mixin配置文件名：
```
    TweakClass: org.spongepowered.asm.launch.MixinTweaker
    TweakOrder: 0
    MixinConfigs: mixins.mymod.json
```
TweakOrder值为`0`可确保尽可能早地注入Mixin转换器。

> **重要注意事项：**FML环境中规定Tweaker通常不能是一个CoreMod。然而，为了克服这个限制，Mixin Tweaker从Jar Manifest文件中读取`FMLCorePlugin`条目，并通过反射将CoreMod类注入FML。

`MixinConfigs`值是一个以逗号分隔的配置文件列表，用于提供给Mixin子系统。

### 使用LiteLoader引导Mixin

Mixin子系统将自动地由LiteLoader引导。只需简单地将`mixinConfigs`键添加到你的`litemod.json`文件。`mixinConfigs`可以是以逗号分割的配置文件列表，也可以是作为字符串的常规JSON数组。

> 译者注：
> 
> <a name="note1"><sup>1</sup></a>转换器（Transformer）是指可修改类字节码达到修改类的目的的类。
>
> <a name="note2"><sup>2</sup></a>转换器链（Transformer Chain）是指一系列以链式结构组合的转换器。
> 
> <a name="note3"><sup>3</sup></a>重映射转换器（Remapping Transformer）是指一种具有特殊功能的转换器，它可以对类名、方法名、字段名进行转换，常用于动态反混淆。
> 
> <a name="note4"><sup>4</sup></a>专用服务端（Dedicated Server）是指仅具有服务器功能的应用程序，区别于客户端的内建服务端。例如MinecraftServer、Spigot等。
>
> <a name="note5"><sup>5</sup></a>LaunchWrapper是Minecraft用于接管游戏启动过程，更便利地修改游戏的启动包装器。
>
> <a name="note6"><sup>6</sup></a>Tweaker是LaunchWrapper用于配置、调整、修改游戏的接口，通常被各API实现。常见的实现有FML和LiteLoader。
>
> <a name="note7"><sup>7</sup></a>第一类（first-class）Tweaker是指在LaunchWrapper启动时便可加载的Tweaker，其类名由命令行参数直接指定，区别于由FML，LiteLoader等加载的间接Tweaker。
