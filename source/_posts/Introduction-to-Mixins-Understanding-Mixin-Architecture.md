---
title: 介绍Mixin——理解Mixin的结构
date: 2018-11-01 08:09:56
tags: [Java, Bytecode, Mixin]
categories: Mixin
---

> 本文翻译自：[Introduction to Mixins Understanding Mixin Architecture](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture)

在开始开发Mixin之前，要使它们产生效果，最重要的是对其基本概念的理解。本文简要介绍了这些概念。尽管您可能熟悉这里所述的所有内容，但我建议至少略读前三个部分，因为它们介绍了我将用来演示如何使用Mixin的示例案例，以及Java和JVM中大量使用Mixin的一些特殊部分。

_**这不是一篇教程** 本介绍并非教程，有关Mixin实现的更多详细信息，请参阅Sponge仓库中的Mixin[示例代码](https://github.com/SpongePowered/SpongeCommon/tree/bleeding/src/example/java/org/spongepowered)。_

> **注意**
> 
> 如果你已经全面了解了字节码、名称绑定，或直白的说你已经知道INVOKESPECIAL到INVOKEVIRTUAL，那么可以[跳到第四节](#4-只有你Mixin能拯救人类)，这将介绍Mixin本身。

### 1. 理解~~Portal~~Mixin - 以实例为例
为了能够想象Mixin是如何工作的，我将给出一个示例。 *注意，这个示例纯粹是为了演示而编写的，与真正的代码库中的名称完全不同！*

在示例中，我们可以看到一个`EntityPlayer`类，它的直接（并且唯一）父类是`Entity`。我们可以用这样的UML风格来表示它：

![图1 - 一个简单的类层次结构](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_0.png)

**图1 - 一个简单的（虚构的）类层次结构**

在Mixin术语中，`EntityPlayer`是**目标类（Target Class）**，Mixin将被应用于该类。

为了充实示例，让我们添加一些假想的字段和方法到想象的示例类中：

![图二 - 一个有成员的简单的类层次结构](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_1.png)

**图二 - 一个有假想字段和方法的简单的类层次结构**

选择这种表示方法是为了凸出哪些成员在类的公开区域，其中公共方法和字段将凸出在类主体之外，因为它们对其他对象是可见的。在使用Mixin时，外界是我们必须记住的一个重要概念。

注意，继承自`Entity`的公共方法*也*是类公共可见区域的一部分，而从父类继承的“幽灵”方法`getHealth`和`setHealth`也存在与类的整体外观中。

在使用Mixin之前，深入了解`this`和`super`这两个Java关键字是非常重要的。这似乎很奇怪，因为任何使用Java超过五分钟的人都会认识这些关键词及其用法，但如果你不想在编写Mixin时抓狂，那么充分理解这两个词的微小差异是至关重要的。

首先来看一下我们假想类中的一些可能的调用和访问：

![图3 - 一些字段和方法的访问](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_2.png)

**图3 - 一些字段和方法的访问**

对于该情况，`this.level`、`this.update()`和`this.food`似乎没什么有争议的，它们看起来都很标准。此外，调用`super.health`和`this.health`，以及从`update`调用`super.onUpdate()`的目的是为了凸出JVM的一个方面，它在编写Java代码时并不明显。

问一下自己下述问题：

 * 用`super`来限定`onUpdate`的调用有什么实际意义，为什么不用`this`？

 * 用`super`来限定`health`的调用有什么实际意义，为什么不用`this`？

毕竟这两种限制在实际中都一样，对吧？

上述两个问题的答案如下：

* `super.onUpdate()` *将始终调用*`Entity`*中的方法，即使子类重写它*，而`this.onUpdate()`将*调用子类中重写的方法*。

* 没有区别，`Entity`中的字段始终从方法`takeDamage`访问，即使子类通过再次声明来“隐藏”字段。

这种行为的根本原因是，用`super`限定的调用和所有字段访问在编译时被**静态绑定（Statically Bound）**，这意味着它们总是引用成员。相反，用`this`限定的访问都是**动态绑定（Dynamically Bound）**，这意味着它们直到实际调用时才解析它们的目标，从而允许子类重写方法并在适当的时候调用它们。

> **注意**
> 
> 除了`super`限定的调用，访问`private`和`static`方法也总是**静态绑定**。
>
> 在字节码中，静态绑定调用是以INVOKESPECIAL和INVOKESTATIC操作符来表示，而动态调用是以INVOKEVIRTUAL操作符来表示。

在开发Mixin时，准确意识到这些关键字的性质是有用的，这也是对Mixin类施加的一些限制的原因，稍后对此进行更多的说明。

### 2. 透镜窥秘
我在上述描述中避免使用*接口*来描述公开可见的成员，以避免与*实际的*接口混淆，因为接口本身在使用Mixin时起着关键的作用。

为了理解接口如何影响我们与类的交互，让我们看看在例子中创建一个包含一些方法的接口，让后通过接口访问这些方法会发生什么。

旁注：*是的，这完全偏离了UML的轨道，但UML对于表示此处的概念并不真的有用，这个框图的**底部**从其他任何对象来看都是“可视区域”，该接口实际上位于公共类“之前”，并给出它的一个子集。*

![图4 - 一个让UML爱好者讨厌的图](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_3.png)

**图4 - 一个让UML爱好者讨厌的图**

这里有一些有用的东西值得注意：

* 首先，必须注意到`Entity`类中的`getHealth`和`setHealth`方法实际上正在实现接口方法，即使`Entity`类不了解`LivingThing`接口，这意味着接口方法中没有任何*特殊*：只要方法签名[<sup>1</sup>](#nb1)与接口中的签名匹配，就认为类方法实现了接口方法。从这里可以清楚地看出接口方法调用是**动态绑定**。

* 我们也没有修改两个类，除了声明它`implements LivingThing`。事实上，如果Java不要求我们包含`implements`子句，那么这个程序结构是合法的，对程序没有任何改变。这告诉我们，如果能够以某种方式偷偷地将`implements`子句插入到目标类中，那么只要接口的方法存在，我们就能在目标类上调用它们。

> <a name="nb1"><sup>1</sup></a> 一个方法的**签名**是它的一组参数*及其返回类型*。例如下述方法：
> ```java
> public ThingType getThingAtLocation(double scale, int x, int y, int z, boolean squash) {
> ```
> 它的签名将会是：
> ```
> (double,int,int,int,boolean)com.mypackage.ThingType
> ```
> 注意，我们将参数放在括号中，最后是返回类型。在实际中，为了节省空间，将会[使用更紧凑的语法](https://www.murrayc.com/permalink/1998/03/13/the-java-class-file-format/#TypeDescriptors)，在字节码中，上述签名是这样的：
> ```
> (DIIIZ)Lcom/mypackage/ThingType;
> ```
> 如果你打算使用*Injector*，你需要熟悉字节码描述符。

### 3. 嘎嘎
我们正在慢慢组装的拼图的最后一块是一个与接口有关的实用的Java语言特性，即你可以将任何对象引用转换为任何接口，编译器很乐意编译它。

例如，假设我们为可以升级的对象创建一个新的接口，叫做`Leveller`，就像这样：

![图5 - Leveller接口](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_4.png)

**图5 - _多么美好的一天啊，嘿嘿_**

下述代码将愉快地编译：
```java
public void method() {
    // 创建一个新的EntityPlayer
    EntityPlayer player = new EntityPlayer();

    // 这将被编译，即使EntityPlayer没有
    // 实际上实现接口，但它在运行时
    // 会抛出一个ClassCastException
    Leveller lev = (Leveller)player;

    // 我们永远不会到达这个代码，但它也将编译好。
    int level = lev.getLevel();
} 
```
上节中我们知道，`EntityPlayer`的方法`getLevel()`**能**愉快地在不改变类的情况下实现接口，但`implements`子句没有显式声明接口这一事实导致在运行时转换失败。如果我们可以在运行时以某种方式添加`implements`字句，那么最终有一种可行的方法以使用接口在Java中实现*鸭子类型（Duck typing）*。

> “实现什么？”<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 可能是你说的

[鸭子类型（Duck typing）](http://en.wikipedia.org/wiki/Duck_typing)是一种在动态类型化语言中使用的隐式类型化方法，它允许基于对象的成员是否存在来访问或调用对象的成员。它的名字来自“鸭子测试（Duck test）”，表达如下：

> 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

换句话说，如果我们只关心一个对象有方法`quack()`和`walk()`，那么对我们而言，它是一个`Duck`，而不关心它是否只是一个非常聪明的`Pigeon`，只要它有这些方法，那它对我们来说就是`Duck`。

如果还不清楚这里说了什么，那么我建议[阅读Wikipedia的条目](http://en.wikipedia.org/wiki/Duck_typing)，因为它详细地涵盖了超出本介绍范围的概念。

那么到目前为止我们知道了什么？

* 我们知道类和接口之间的关系非常脆弱，只需稍加修改就可以以多种方式做对我们有利的事情。

* 我们知道可以利用Java中的**动态绑定**编写能通过编译的代码（即使它不能运行），并以某种方式将`implements`子句添加到目标对象上是这项工作的关键。

* 我们知道在编译时，使用`super`关键字的父类调用是**静态绑定**，这意味着在指定`super`时，我们需要额外考虑，我们指定的是*什么*。

最后要考虑的是，当类*不*实现接口时会发生什么。让我们把另一个名为`setLevel()`的方法添加到我们的示例接口`Leveller`中：

![图6 - 添加setLevel()](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_5.png)

**图6 - 添加setLevel()**

将第二个方法添加到接口中会增加另一个 - 不同的 - 运行时错误，在本例中为`AbstractMethodError`。

```java
public void method() {
    EntityPlayer player = new EntityPlayer();

    // 假设我们可以运行时将接口声明添加到EntityPlayer类，允许这个操作成功。
    Leveller lev = (Leveller)player;

    // 该语句会在运行时抛出AbstractMethodError，
    // 因为setLevel(I)在EntityPlayer及其任何父类中都没有定义。
    lev.setLevel(10);
} 
```

理解了Java和JVM的这些方面，让我们来看看Mixin它自己。

### 4. 只有~~你~~Mixin能拯救人类
那么现在我们知道Mixin必须完成的基本任务，以便使我们可以使其他的对象嘎嘎：

1. 让我们在运行时将我们所选的接口应用到**目标类**
2. 让我们为接口中声明但**目标类**中不存在的任何方法插入一个方法*实现*

首先让我们看看如何声明一个Mixin类，以`EntityPlayer`作为它的**目标类**：

```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity {
}
```

是的，就是这么简单。使用`@Mixin`注解将这个类定义为一个Mixin类，并指定我们想应用它的**目标类**。但要注意：

* Mixin类使用`abstract`修饰符标记。虽然这不是必须的，但当在IDE中使用Mixin时它会很有用，因为它意味着终端用户不能编写试图实例化Mixin类的代码，这会在运行时导致错误。它还避免了必须实现任何声明的接口中的每一个方法的要求（Java编译器强加的），这是Mixin的主要目的之一。

* Mixin类继承了`Entity`，这是与我们的**目标类**相同的超类。这是很重要的，用以保持任何**静态编译**的语义编译到我们的Mixin类中。稍后再详细说明。

如果我们现在在运行时包含这个Mixin并运行游戏，那么将应用Mixin但绝对不会更改任何内容，这是因为我们实际上没有在Mixin中声明任何内容。让我们来看看如何实现上述目标**1**，并使用Mixin在**目标类**上添加新接口：

```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing {
}
```
就是这样！当Mixin被处理时，在Mixin上声明的任何接口都被应用到**目标类**。让我们看看当前的类层次结构：

![图7 - Mixin层次结构（应用前）](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_6.png)

**图7 - Mixin层次结构（应用前）**

虽然这个图代表了我们将创建的类的实际层次结构，但它实际上更用于（并且在一些更复杂的情况下，至关重要）*认识到Mixin**不是真正的类**。*在运行时，Mixin将被应用到**目标类**，因此，认为Mixin*存在于**目标类内***反而更有利于良好的思考过程。

在Mixin应用之后，新的类层次看起来像这样：

![图8 - 类层次结构（应用后）](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_7.png)

**图8 - 类层次结构（应用后）**

如你所见，目标类现在实现了`LivingThing`接口，现在允许我们的鸭子类型按所想那样使用了：
```java
public void method() {
    EntityPlayer player = new EntityPlayer();

    // 应用了Mixin之后，转换成功
    LivingThing living = (LivingThing)player;

    // 因为转换成功了，所以我们可以传递对象
    // 到其他需要应用LivingThing的地方
    // 如下所示
    if (this.isAlive(living)) {
        // 万岁
    }
}

public boolean isAlive(LivingThing living) {
    // 我们现在可以很好地调用getHealth()方法，因为该方法
    // 存在并可通过LivingThing接口访问
    int health = living.getHealth();
    return health > 0;
} 
```
由于我们已经实现了第一个目标，现在可以成功地向**目标类**应用新的接口，因此让我们来看看第二个目标：

* 让我们为接口中声明但**目标类**中不存在的任何方法插入一个方法*实现*

我们首先让我们的Mixin类实现`Leveller`接口，该接口声明当前未在我们的**目标类**及其任何父类中实现方法：
```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing, Leveller {
}
```
生成以下类层次：

![图9 - Mixin类层次（应用前）](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_8.png)

**图9 - Mixin类层次（应用前）**

因为我们的Mixin类是`abstract`，该代码将很好地通过编译，但是在运行时任何对`setLevel()`方法的调用都会抛出如上所述的`AbstractMethodError`。我们可以在Mixin自身中定义`setLevel()`方法来解决这个问题：
```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing, Leveller {

    @Override
    public void setLevel(int newLevel) {
        // TODO 实现该方法
    }
}
```
![图10 - 添加一个方法到Mixin](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_9.png)

**图10 - 添加一个方法到Mixin**

现在，当应用Mixin时，新方法也将被添加到**目标类**：

![图11 - 类层次结构（应用后）](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_10.png)

**图11 - 类层次结构（应用后）**

我们修改的目标类现在完全实现了所有声明的接口，我们可以看到想目标类添加新方法是多么容易。但目前我们的新方法实际上没有做任何事情，我们将在下一节中看到如何修复。

### 5. 点燃蜡烛将投下Shadow
因此，现在我们有办法将新方法注入**目标类**，但是在实现新注入的方法体时，我们就很快遇到一个问题：在理想情况下，我们希望新的`setLevel()`实现能够访问`EntityPlayer`中的`level`变量。但有一个问题是……它不能。

![图12 - 不可能的访问](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_11.png)

**图12 - 不可能的访问**

我们不能访问**目标类**的成员，因为在实际应用Mixin之前，字段不存在！因为Mixin类的父类是`Entity`，如果字段是`protected`，它甚至没用：对Java编译器而言，字段是不可视的。

但**我们知道**当Mixin被应用时，字段**将在那里**，我们需要的是一种方法告诉Java*“嘿，这个字段**将**会存在，让我访问它”*。幸运的是，Mixin提供了一个机制，通过`@Shadow`注释做到这一点：

```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing, Leveller {
    
    @Shadow
    private int level;
    
    @Override
    public void setLevel(int newLevel) {
        // 引用上面的影子字段，但将引用
        // 应用Mixin后的真正的字段
        this.level = newLevel;
    }
}
```
`@Shadow`注释在Mixin中创建一个“虚拟字段”，它反映了**目标类**的对应部分：

![图13 - 我和我的影子](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_12.png)

**图13 - 我和我的影子**

还可以将`@Shadow`使用在方法上，一遍调用只在目标类中定义的方法，例如，在设置等级后立刻调用`update()`方法，我们可以轻松的影射方法，然后从新的`setLevel()`方法体中调用它：

```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing, Leveller {
    
    @Shadow
    private int level;
    
    @Shadow
    private void update() {}
    
    @Override
    public void setLevel(int newLevel) {
        // 设置等级的值
        this.level = newLevel;
        
        // 调用影射的方法以更新实体状态
        this.update();
    }
}
```
我们通常将影子方法声明为`abstract`，只是为了避免编写方法体，但很显然，我们不可能将`private`与`abstract`同时声明，所以我们只是用空方法体声明影子方法。

![图14 - 影射万物](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_13.png)

**图14 - 影射万物**

### 6. 它是鸟吗？是飞机吗？不，它是父类！
旅途的最后一站是关于Mixin的基本特性，简要介绍如何在Mixin中处理父类的访问。首先，我们需要理解为什么一个Mixin类被声明为与**目标类**相同的超类。

首先让我们快看看当前的类层次结构：

![图15 - 游戏状态](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_14.png)

**图15 - 游戏状态**

请记住，从第一节开始使用`super`关键字的调用都是**静态绑定**的。在我们的Mixin类上下文中，如果我们如**图15**所示调用`super.onUpdate()`，那么生成的字节码将具体地引用`Entity`类中的`onUpdate`方法。

当Mixin与**目标类**具有相同的父类时，这正是我们想要的。然而，实际上Mixin可以继承*目标类层次结构上的任何类*，直到并包括`Object`。

让我们假设一下，`EntityPlayer`不是直接从`Entity`继承的，而是从中间的一个类`EntityMoving`，而Mixin类仍然可以直接继承`Entity`：

![图16 - 继承层次结构 - 注意：此图是故意错误的！](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_15.png)

**图16 - 继承层次结构 - 注意：此图是故意错误的！**

看看这个新的层次结构，现在很明显为什么`super.onUpdate()`将*出现*在Mixin类中调用`Entity`的方法，但这里很重要的一点是，**忽略IDE（可能还有常识）告诉你的，并记住Mixin的关注点永远在 _目标类_**！

这里的问题是，中间类`EntityMoving`已经重写了`onUpdate`，并且类的功能范围使得在超类中调用`onUpdate`实际上会导致不一样的行为。当我们在Mixin中调用`super.onUpdate()`时，它**必须**具有**相同**的语义，就像**从目标类**调用同一个Java语句一样，并且**确实如此**。

* 为了保持你键入到Mixin中的Java代码的语义一致性，Mixin转换器在应用时更新Mixin类中所有的**静态绑定**。这意味着在上述例子中，调用`super.onUpdate()`将正确地调用`EntityMoving`中的方法。

* 这并不影响`this`关键词的语义。对于`protected`和`public`方法，它们总是使用**动态绑定**，因此总是调用适当的子类方法。

> 为了实现该技术，转换器将处理Mixin中所有的INVOKESPECIAL操作符，并分析目标类的父类层次结构，以找到该方法的最特化的版本。该过程开销很高，并且只在“分离的”Mixin（那些父类与目标类的父类不同的Mixin）上执行。为了避免这种处理步骤，建议尽可能地将Mixin类与它们的目标类具有相同的父类。

![图17 - 最终层次结构](./Introduction-to-Mixins-Understanding-Mixin-Architecture/mixin_tut_16.png)

**图17 - 最终层次结构（Mixin应用后）**

如你所见，将Mixin应用到目标类之后，将`super.onUpdate()`调用的语言更新为与**目标类**一致，并且一切都再次工作良好。

### 7. 圆满完成

虽然本介绍涵盖了Mixin的基本知识，但是还有很多方面需要探讨，尤其是在**目标类**在使用之前会被混淆的生产环境中工作时。

#### 更多Mixin文章（即将到来）

* [Introduction to Mixins - The Mixin Environment](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment)
* [Introduction to Mixins - Overwriting Methods](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Overwriting-Methods)
* [Resolving Method Signature Conflicts](https://github.com/SpongePowered/Mixin/wiki/Resolving-Method-Signature-Conflicts)
* [Obfuscation and Mixins](https://github.com/SpongePowered/Mixin/wiki/Obfuscation-and-Mixins)
* [Advanced Mixin Usage - Soft Implementation](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Soft-Implementation)
* [Advanced Mixin Usage - Using Injection](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Using-Injection)