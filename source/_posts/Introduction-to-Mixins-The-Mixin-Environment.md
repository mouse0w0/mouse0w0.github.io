---
title: 介绍Mixin——Mixin环境
date: 2018-11-01 17:18:21
tags: [Java, Bytecode, Mixin]
---
> 本文翻译自：[Introduction to Mixins The Mixin Environment](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment)

现在我们已经了解了Mixin的[基本特性](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture/)（提示：如果你还没有读过，请返回并首先阅读它！），让我们快点绕开，来了解如何使用我们的新创建的Mixin进入游戏环境。

#### Mixin是如何工作的

正如我在上篇[文章](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture/)中所提到的，对于大多数情况，最好记住**Mixin不是严格意义上的类**，也不是在运行时加载的类，而是使用[ASM](http://asm.ow2.org/)的Tree API解析的原始字节码。这种方法生成一个基于节点的基础字节视图，然后可以通过Mixin转换器合并到目标类中。

Mixin本身是独立于常规类，通过转换器链传送的，这是为了允许任何重映射转换器在Mixin与目标类组合之前完成它们的工作。因此，Mixin转换器在任何重映射转换器的*下游*是很重要的，这是通过在转换器链中插入代理转换器来内部处理的。

![Mixin转换器链](mixin_transformer_chain.png)

Mixin需要在类开始载入之前进行处理，以便识别它们的**目标类**，进行[**层次结构验证**](https://github.com/SpongePowered/Mixin/wiki/About-Hierarchy-Validation-in-Mixins)，并为方法和字段解析任何更新的**静态绑定**。你希望使用的Mixin应该在配置文件中指定，你的CoreMod、Tweaker、Jar Metadata或litemod.json将指定模组的Jar内的资源位置。

> 配置路径被作为*资源*加载，因此可以放在Jar任何包中。重要的是，资源路径必须是相对的，**不要**在文件路径上预先准备一个主斜杠。
>
> 例如，如果您的配置文件在包`com.somepackage`中，则应该将配置文件的路径指定为`com/somepackage/myconfig.json`。

每个**配置**定义一个**Mixin集合**，并且对于你的应用程序，可能会有多个**Mixin集合**。当你的Mixin针对不同的*环境*（见下文）时，只*需要*分离Mixin到不同的集合中。然而，有时也希望为组织结构而将Mixin拆分成集合。

### Mixin配置文件

除了定义**Mixin集合**之外，Mixin配置文件还定义了集合的附加属性。

首先，用四个键来定义集合本身：

* `package`为这组Mixin定义了父包（这很重要，因为该包和所有子包在运行时都将被排除在`LaunchClassLoader`之外）。
* `mixins`为该配置定义了位于*父包*内的*Mixin“类”列表*，每个Mixin“类”都相对于*父包*指定，可以使用子包。此列表中的每一个条目都将应用于客户端和专用服务端侧。
* `client`定义了一组**仅**应用于**客户端**侧的Mixin。
* `server`定义了一组**仅**应用于**专用服务端**侧的Mixin。

然后可以定义附加属性：

* `refmap`定义该集合的**引用映射（Reference Map）**文件名（关于如何使用Mixin处理混淆之后再做更多讨论）。
* `priority`定义了这个Mixin集合相对于其他配置的**优先级（Priority）**。
* `plugin`是可选的用于Mixin配置的**伴随插件**的类名，该类可以在运行时以编程方式调整Mixin配置，参见~~Mixin伴随插件~~（尚未完成）。
* `required`定义了Mixin集合是否**必须**。如果单个Mixin未能应用，则应将其视为整个游戏处于失败状态，此时*required*标志应设为`true`。
* `minVersion`在当Mixin集合使用一些在特定版本添加的Mixin功能时设置。它可以省略版本不可用的Mixin集合
* `setSourceFile`是Mixin处理器用Mixin类的源代码文件覆盖目标类中的`source file`属性。这在调试Mixin时很有用。
* `verbos`将所有`DEBUG`级别的日志消息等级调整为Mixin集合的`INFO`级。这也可以通过`mixin.debug.verbose`[系统属性](https://github.com/SpongePowered/Mixin/wiki/Mixin-Java-System-Properties)在全局范围内启用。

需要注意的最重要的一点是，Mixin“类”必须位于一个包中，且其中没有你的项目的其他类（这包括Mixin主包的子包），因为启动时整个包将被从转换类加载器中被排除。

### 游戏生命周期与你

为了理解Mixin如何与游戏底层交互，首先理解运行在`LaunchWrapper`中的游戏生命周期是至关重要的。

*LaunchWrapper*接管了游戏的正常启动过程，以便允许*Tweaker*修改游戏。每个*Tweaker*可以提供*类转换器（Class Transformer）*，这能够在游戏类加载时修改它。*Tweaker*在命令行中指定，并由`Launch`类逐个加载并初始化，该类构成了*LaunchWrapper*启动逻辑的核心。

像FML和LiteLoader这样的*Tweaker*还能够合作地加载和注入它们找到的其他*Tweaker*，因此`Launch`在一个循环中初始化*Tweaker*，直到不再继续提供额外的*Tweaker*。我们可以想象`Launch`中的执行流程如下：

![启动生命周期](mixin_env_0.png)

这个初始化阶段*必须*在游戏开始加载之前完成，这是绝对必要的，否则因为在*Tweak*初始化周期中晚点注册的*Transformer*可能没有机会处理它需要的类，并且转换器链也可能不完整。

在初始化阶段发生的事情是：

* 像FML和LiteLoader这样的Tweaker寻找Mod，并且注入它们的转换器和任何它们找到的额外的Tweaker。
* FML “Core Mod” （又叫载入时插件（Loading Plugin））被初始化并可以注册它们自己的转换器。

由于Mixin子系统必须在这个早期阶段初始化（在游戏类加载之前），所以它必须作为CoreMod或Tweaker加载，参阅下面的引导部分。

![启动生命周期](mixin_env_1.png)

一旦*Tweaker*初始化完成，`Launch`调用游戏的原始`main()`方法，该方法启动游戏加载过程。在此之上，转换器链已经完成，因此转换器链可以通过注册的转换器加载和处理游戏类。游戏进入其主循环，按需加载类，并有转换器链（包括Mixin）处理。

上述内容应该清楚的是，游戏执行因此被分成两个截然不同的*阶段*，*Tweaker*在**预初始化阶段（pre-init phase）**被初始化，以及类似于正常的游戏生命周期的**默认阶段（default phase）**，该阶段可不在Wrapper内运行。有必要了解这些阶段及其与Mixin子系统的关系。

#### 这只是我经历的一个阶段

在通常情况下，我们的Mixin设置将只处理在**默认**阶段加载的类，并且事情非常简单。在这个阶段，我们已知：

* 转换器链是完整的
* 游戏类能被安全的载入

这意味着加载和转换游戏类字节码并应用Mixin是安全的，加载以便应用那些Mixin的必须的*类元数据*也是安全的（请记住，从上开始，Mixin的验证和预转换在启动时一次性完成）。如果我们过早地生成元数据（例如，在初始化之前的阶段），那么就有可能没有注册关键的转换器！这将导致Mixin字节码不完整或无效。

_那么为什么要担心**预初始化**？_

简单的答案是：*“所以我们可以组合到其他Tweaker提供的核心类，特别是FML”*

像*Sponge*这样的平台的要求之一是，有时必须以某种方式挂钩到底层平台，如果转换发生在**Default**阶段时，核心类已经加载（这已经超出了转换器的范围）。然而，正如我们确定的，在**预初始化**阶段加载我们的游戏相关的Mixin将不会起作用，因为需要的转换器将不存在。

#### 关于环境

然后，在Mixin处理器中，我们确认将处理分割到_**环境**_（每个阶段一个）中，然后你可以针对所期望应用的*环境*分割你的Mixin集合。典型的Mixin情景如下方式解决：

* 一个只组合游戏类的Mixin集合应该该**默认**环境中注册
* 当需要在两个阶段应用Mixin时，两个Mixin集合（配置）应该在一个集合中指定为**预初始化**Mixin，另一个集合则指定为**默认**Mixin。

#### 跃跃欲试

值得注意的一件事是，作为一个*Tweaker*，FML合作地加载它的CoreMod，并使用代理*Tweaker*将它们重新注入到启动生命周期中。因为我们对启动顺序可能有点微妙的敏感，因此值得注意的是，*“第一类（first-class）”*Tweaker和CoreMod初始化期间产生的间接Tweaker有细微区别。

Mixin库包括一个第一类的Tweaker，它被设计为在尽可能早的时候注册并开始处理，这在下图中由 **(1)** 标记。正常的CoreMod启动发生于 **(2)** 处的第一次循环，然后游戏加载开始于 **(3)**。

![启动生命周期](mixin_env_2.png)

如果需要在**预初始化**组合到类中，则建议使用第一类Tweaker。

### 使用FML CoreMod引导Mixin

> **注意：在生产环境中通过CoreMod引导Mixin目前是不可能的，要在发布版本中使用Mixin和Forge，你必须使用下述的调整方法。**但是，你可能希望通过CoreMod在开发工作空间中加载Mixin，这是可能的，但它必须在生产环境中使用Tweaker：

使用CoreMod在开发环境中引导Mixin：

* 在你的CoreMod的构造函数中，使用以下代码初始化Mixin环境并提供Mixin配置文件的名称：
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

Mixin库内建有第一类*Tweaker*，你可以通过在你的Mod的Jar元数据中指定Tweaker类来使用。为此，只需像如下所示简单地指定Tweaker类名和你的Mixin配置文件名：
```
    TweakClass: org.spongepowered.asm.launch.MixinTweaker
    TweakOrder: 0
    MixinConfigs: mixins.mymod.json
```
TweakOrder值为`0`可确保尽可能早地注入Mixin转换器。

> **重要注意事项：**在FML环境中，规定Tweaker通常不能是一个CoreMod。然而，为了克服这个限制，Mixin Tweaker从Jar Manifest文件中读取`FMLCorePlugin`条目，并通过反射将CoreMod类注入FML。

`MixinConfigs`值是一个以逗号分隔的配置文件列表，用于提供给Mixin子系统。

### 使用LiteLoader引导Mixin

Mixin子系统将自动地由LiteLoader引导。只需简单地将`mixinConfigs`键添加到你的`litemod.json`文件。`mixinConfigs`可以是以逗号分割的配置文件列表，也可以是作为字符串的常规JSON数组。

