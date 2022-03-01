---
title: 在Minecraft Forge中使用Mixin
date: 2022-03-01 23:00:00
tags: [Java, Forge, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Mixins on Minecraft Forge](https://github.com/SpongePowered/Mixin/wiki/Mixins-on-Minecraft-Forge)

Mixin现在作为库与[Minecraft Forge](https://www.minecraftforge.net/)一同发布，这意味着不再需要像传统版本那样，将Mixin嵌入到模组Jar中。实际上，这样做会引发问题。

如[混淆与Mixin](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Obfuscation-and-Mixins)一章中所述，当你为生产环境构建模组时，某些Mixin功能需要特殊处理才能跨越*混淆边界*。本指南说明了如何在Forge UserDev环境中使用Mixin功能，以及如何配置构建，以便为Mixin正确生成混淆信息。

本指南假定你已经创建了一个**包（Package）**来包含你的Mixin和一个[Mixin配置](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment#mixin-configuration-files)来声明Mixin和选项。

为了配置要在开发中运行的项目，我们将对`build.gradle`进行如下更改，并管理混淆：

1. 添加[MixinGradle](https://github.com/SpongePowered/MixinGradle)插件。
2. 添加Mixin**注解处理器（Annotation Processor）**依赖。
3. 通过`mixin`闭包（Closure）为**MixinGradle**配置选项。

> 在本指南中，将假定你的Mixin配置名为`mixins.mymod.json`，而refmap并命名为`mixins.mymod.refmap.json`，并且你只使用了`main`源代码集（SourceSet）。

## 步骤 1 - 添加MixinGradle插件

打开你的`build.gradle`文件并在文件顶部找到`buildscript`块。在`buildscript`块内的`repositories`块的正下方是`dependencies`块，它指定了构建脚本本身（而不是项目）所需的依赖关系。我们将添加**MixinGradle**依赖项，以便加载插件：

```groovy
buildscript {
    repositories {
       ...
    }
    dependencies {
        classpath group: 'net.minecraftforge.gradle', name: 'ForgeGradle', version: '5.1.+', changing: true
        // 添加 MixinGradle 插件
        classpath 'org.spongepowered:mixingradle:0.7.+'
    }
}
```

现在已经注册了依赖项，我们可以将插件应用到我们的构建中。在`buildscript`块的正下方，你将看到已经应用了三个插件。我们将在现有插件下面添加**MixinGradle**：

```groovy
apply plugin: 'net.minecraftforge.gradle'
// Only edit below this line, the above code adds and enables the necessary things for Forge to be setup.
apply plugin: 'eclipse'
apply plugin: 'maven-publish'
// 应用 MixinGradle 插件
apply plugin: 'org.spongepowered.mixin'
```
## 步骤 2 - 添加Mixin注解处理器

**MixinGradle**的工作是*配置*Mixin注解处理器（AP），但是AP不是自动应用的，我们需要将其作为依赖项添加。AP依赖项应该与项目中使用的版本相同或更高（如果不确定使用的是哪个版本的Mixin，请参阅本指南末尾的部分）。

要将Mixin AP依赖项添加到项目中，请找到该项目的`dependencies`块。**这_不是_Gradle文件顶部`buildscript`块中的`dependencies`块，而是文件下面更大的`dependencies`块。**在本指南中，我们假定版本为`0.8.5`（撰写本文时的当前版本）。

它当前应该包含`minecraft`依赖项和一组注释，以及您手动添加的任何其它依赖项。`main`的AP配置称为`annotationProcessor`。将Mixin AP依赖项添加到`dependencies`块：

```groovy
dependencies {
    // Specify the version of Minecraft to use. If this is any group other than 'net.minecraft', it is assumed
    // that the dep is a ForgeGradle 'patcher' dependency, and its patches will be applied.
    // The userdev artifact is a special name and will get all sorts of transformations applied to it.
    minecraft 'net.minecraftforge:forge:1.17.1-37.0.70'

    // 应用 Mixin AP
    annotationProcessor 'org.spongepowered:mixin:0.8.5:processor'
```

> 注意，AP依赖项含有额外的*分类词*`processor`。这是一个特殊的臃肿Jar（Fat Jar），其中包含Mixin AP所需的上游依赖项，因此你不必手动指定它们。

### 步骤 2.1 - 如果你有其它含有Mixin的源代码集

*如果你只有一个源代码集（SourceSet）`main`，你可以跳过本节。*

额外的源代码集将有相应的*Gradle配置*，用于定义它们的依赖关系。如果你有其它含有Mixin的源代码集，那么每个源代码集都需要自己的**参考映射（Reference Map）**（下面将详细介绍），每个源代码集都需要配置Mixin AP依赖项。

让我们假定你有额外的名为`client`和`api`的源代码集。相应的配置名称将是`clientAnnotationProcessor`和`apiAnnotationProcessor`，将Mixin AP添加到其中：

```groovy
dependencies {
    // Specify the version of Minecraft to use. If this is any group other than 'net.minecraft', it is assumed
    // that the dep is a ForgeGradle 'patcher' dependency, and its patches will be applied.
    // The userdev artifact is a special name and will get all sorts of transformations applied to it.
    minecraft 'net.minecraftforge:forge:1.17.1-37.0.70'

    annotationProcessor 'org.spongepowered:mixin:0.8.5:processor'
    clientAnnotationProcessor 'org.spongepowered:mixin:0.8.5:processor'
    apiAnnotationProcessor 'org.spongepowered:mixin:0.8.5:processor'
```

## 步骤 3 - 配置MixinGradle

**MixinGradle**提供了一个名为`mixin`的*扩展*，允许我们配置插件的选项，**MixinGradle**然后将这些设置应用到适当的位置。要配置扩展，请指定扩展名和包含我们设置的闭包。这可以放在构建中的任何地方，但我建议将其放在`sourceSets`或`dependencies`块之后：

```groovy
mixin {
    // MixinGradle 设置
}
```

### 重要设置

**MixinGradle**可以配置AP选项，但我们需要告诉它要为每个编译任务生成的refmap的名称，这必须与配置JSON中的refmap名称相对应。

> 请注意，同一源代码集中的配置都应该指定相同的refmap名称，而不管它们包含哪些Mixin。这是因为refmap与SourceSet的compile任务耦合在一起，而不关心你的配置中的Mixin的组织方式。

我们还可以指定Mixin配置的名称，这将执行两个任务：

* 配置将被注入到我们所有的*运行配置*中。
* 配置名称将添加到所有混淆Jar的Manifest中（在`MixinConfigs`键中）。

让我们将`main`源代码集的refmap名称和配置名称添加到`mixin`闭包中：

```groovy
mixin {
    // MixinGradle 设置
    add sourceSets.main, 'mixins.mymod.refmap.json'
    config 'mixins.mymod.json'
}
```

### 实用设置
“mixin”闭包也可用于配置选项，如mixin系统属性（用于开发运行）和*Mixin注释处理器*的选项。例如，我建议设置“mixin”。调试。冗长的和混合的。调试。在开发过程中进行出口。为了方便起见，我们可以在“mixin”闭包中设置这些属性：

`mixin`闭包也可用于配置选项，如Mixin[系统属性](https://github.com/SpongePowered/Mixin/wiki/Mixin-Java-System-Properties)（用于开发运行）和*Mixin注解处理器*的选项。例如，我建议在开发期间设置`mixin.debug.verbose`和`mixin.debug.export`。为了方便起见，我们可以在`mixin`闭包中设置这些属性：

```groovy
mixin {
    // MixinGradle 设置
    add sourceSets.main, 'mixins.mymod.refmap.json'
    config 'mixins.mymod.json'

    debug.verbose = true
    debug.export = true
}
```

> 在更改影响运行配置的任何设置后，请记住使用`genEclipseRuns`，`genIntellijRuns`或`genVSCodeRuns`任务重新生成运行配置！

### 步骤 4.1 - 如果你有其它含有Mixin的源代码集

*如果你只有一个源代码集（SourceSet）`main`，你可以跳过本节。*

除了*Gradle配置*，项目中的每一个源代码集都会获得一个对应的*Java编译任务*，它以源代码集命名。由于每个源代码集都是单独编译的，AP每次都会单独运行，因此每个源代码集都需要一个单独的*参考映射*（refmap）。基于上面的示例，让我们为`client`和`api`源代码集添加refmap名和配置：

```groovy
mixin {
    add sourceSets.main, 'mixins.mymod.refmap.json'
    add sourceSets.client, 'mixins.mymod.client.refmap.json'
    add sourceSets.api, 'mixins.mymod.api.refmap.json'

    config 'mixins.mymod.json'
    config 'mixins.mymod.client.json'
    config 'mixins.mymod.api.json'
}
```

## 注解处理器选项

**MixinGradle**使用所有必需的选项配置AP。但是，Mixin AP选项可以通过`mixin`闭包进行操纵。以下是一些实用选项：

<dl>
  <dt>boolean <tt>disableTargetValidator</tt></dt>
  <dd>禁用目标验证器（验证Mixin目标是否合理（例如，目标层次结构中存在超类））</dd>
  <dt>boolean <tt>disableOverwriteChecker</tt></dt>
  <dd>禁用重写方法检查器，它用来确保<tt>&#64;Overwrite</tt>方法的JavaDoc含有<tt>&#64;author</tt>和<tt>&#64;reason</tt>标记</dd>
  <dt>String <tt>overwriteErrorLevel</tt></dt>
  <dd>为重写方法检查器设置错误级别（默认为<tt>warning</tt>，可以被设置为<tt>error</tt>）</dd>
</dl>
<dl>
  <dt><tt>quiet</tt></dt>
  <dd>抑制来自AP的突出消息和信息输出</dd>
  <dt><tt>extraMappings</tt></dt>
  <dd>指定要提供给AP的附加（自定义）TRSG映射文件的名称，这可用于主映射文件中未指定的映射项</dd>
</dl>

```groovy
mixin {
    // AP 设置
    disableTargetValidator = true
    overwriteErrorLevel = 'error'
    quiet
    extraMappings file("my_custom_srgs.tsrg")
}    
```

## 带有选项的`mixin`闭包示例

下面是一个完整的例子，展示了一旦我们配置了所有内容，`mixin`闭包可能会是什么样子：

```groovy
mixin {
    // 每个源代码集的refmap
    add sourceSets.main, 'mixins.mymod.refmap.json'

    // 配置将添加到每个运行配置和Jar
    config 'mixins.mymod.json'

    // 指定开发环境运行配置的选项
    debug.verbose = true
    debug.export = true
    dumpTargetOnFailure = true

    // 注解处理器的选项
    quiet
}
```

## 如何找到Mixin版本

如果你不确定项目中使用的是哪个版本的Mixin，可以通过以下几种方式找到它：

1. 在IDE中，检查项目的依赖项列表。例如，在Eclipse中，你可以在`Project and External Dependencies`容器中查看。
2. 使用`gradle dependencies`任务将项目的依赖关系树发送到控制台，并使用`find`（在Windows上）或`grep`（在Linux上）对其进行过滤：

       gradlew --console=plain dependencies | find "mixin"

   这应该会向控制台发出几行代码，辨识Mixin依赖版本应该不会太难。
3. 在开发环境中运行游戏，然后搜索`debug.log`，查找`mixin subsystem version`。