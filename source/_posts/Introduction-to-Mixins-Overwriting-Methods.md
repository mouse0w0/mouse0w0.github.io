---
title: Mixin介绍——重写方法
date: 2018-11-07 11:47:18
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[
Introduction to Mixins Overwriting Methods](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Overwriting-Methods)

到目前为止，我们讲解的Mixin功能本质上都是**增添**，并且不会从根本上改变目标类的原始行为。但在两种情况下，我们可能希望Mixin修改或替换目标类中的内容：

1. 我们希望更改现有方法的行为。
2. 我们希望在已经存在的目标类中定义一个访问器，但是它的名称在被混淆后可能发生变化。

这两种情况都需要我们能够**重写**目标类中的某些部分。

### 1. 房间里的大象 - 重写

*重写（Overwrite）*是Mixin中最微妙的功能，通常应尽量使用[回调注入器（Callback Injectors）](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Rerouters)、[重路由器（Rerouters）](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors)或其他Mixin代码注入功能。

让我们花一点时间来回顾我们在[本教程的第一部分](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Understanding-Mixin-Architecture#6-is-it-a-bird-is-it-a-plane-no-its-superclass)中的例子，使用Mixin将`setLevel`方法添加到目标类中：

![](mixin_tut_19.png)

Mixin中含有没有多余修饰的附加方法，并且该方法将被添加到目标类中。在Mixin应用之后，该方法将存在于目标类中，就好像它一直存在一样。我已经用![](mixin_tut_21.png)标记了Mixin方法，以便于合并时更容易找到它：

![](mixin_tut_20.png)

因此，下一个问题是：*如果我们在Mixin中声明一个 __已经存在于目标类中的方法__，会发生什么？*

答案是，**_Mixin_ 方法完全重写了 _原始_ 方法**。

#### 1.1 声明重写方法

让我们来看一个简单的例子。我们将假定`EntityPlayer`中的`getLevel()`方法不按接口所需的方式运行。也许按照接口约定，调用`getLevel()`应该总是返回非负值，但`EntityPlayer`的内部代码不能避免负等级，我们希望避开这种情况。

我们将为`getLevel()`定义一个遵守接口约定的新方法体：

```java
@Mixin(EntityPlayer.class)
public abstract class MixinEntityPlayer
    extends Entity
    implements LivingThing, Leveller {

    @Shadow
    private int level;

    /**
     * 该方法重写了目标类中的getLevel方法
     * 并确保它返回非负值
     */
    public int getLevel() {
        return Math.max(this.level, 0);
    }

    @Override
    public void setLevel(int newLevel) {
    
        //... 以下代码略.
```
现在，当应用Mixin时，在Mixin中定义的`getLevel()`方法将重写现有的对应部分：

![](mixin_tut_22.png)

最后类的结构与之前没有什么不同，但包含了我们在`getLevel()`方法中定义的新逻辑。

#### 1.2 能力越大，责任越大

早就应该指出，**重写不是万能的**，它有许多相当明显的缺点，在选择使用它时应该小心。缺点包括但不限于：

* 重写完全清除之前应用于该方法的任何转换。这意味着任何其他使用转换器来修改目标方法的Mod都将清除它们的修改。这可能导致另一个Mod，甚至整个游戏变得不稳定。
* 和其他Mod一样，其他重写相同方法的Mixin也会发生冲突。拥有最高**优先级**的Mixin将取胜，并且只有来自该Mixin的重写才会实际出现在目标类中。如果稍后有Mixin试图重写拥有较高优先级的Mixin重写的方法，就会发出警告。
* 重写更复杂的方法可能导致重写“过时”。例如，如果你决定用原始代码的修改版重写一个方法，那么就应该负责确保该代码在你的Mixin中与其目标保持一致。你可以用[约束](#3-约束)（见下文）来帮助你做到这点。
* 使用重写可能会导致过早秃头和在交通中大声喊叫的倾向。

因此，你应该慎重考虑是否使用重写。经过考究，一些适合使用重写的地方是：

* **简单的方法，例如访问器**
在这些场景中使用重写以用额外逻辑修饰访问器，可以快速高效地添加诸如参数验证之类的功能。由于获取器和设置器不多变，使用重写来修改它们是快速和直接的。然而，你仍然应该考虑使用[注入器（Injector）](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors)是否更合适。

* **快速原型**
重写可以方便的另一个地方是，当原型改变，你打算稍后更改使用[回调注入器（Callback Injector）](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors)的方法时。在Mixin中复制原始方法可以快速简单地在开发过程中创建一个“补丁”，尤其是当试图确定最好的方法来改变以适合你需要的方法的行为时。

* **注入器根本无力做到的情况**
虽然[回调注入器](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors)非常强大，但它们的作用域是有限的，并且你偶尔会遇到它们不符合要求的情况，尤其是在非常复杂或庞大的方法中。与其他情况一样，当选择重写时，应该极度不情愿，但有时这是不得已而为之的。

* **小心那根绳子**
重写就像是拿在你手里的一根很长的绳子。通过设立审核制度来管理正在进行的任何重写，确保重写不会回头伤害你。我建议你至少：
 
 * **当**有人添加重写时，用注释解释**所有**重写的方法，解释**为什么**要使用重写，**谁**添加了重写。在持续的基础上回顾这些注释，以确保重写仍然是必要和有意义的。
 * 当使用“复制和修改”方式重写时，以原始方法体作为起点，在方法体中**注释你做出的修改**。如果目标方法更改，这将更容易提取和合并更改。
 * 解释任何用于原型的重写，或“将其转换为注入器”，以便它们不会在代码库中停留过长的时间。
 * 使用[约束](#3-约束)来为重写添加一个健全的检查，下面的章节中提供更多关于这的详细信息。

#### 1.3 讨厌的混淆边界

你可能想知道[上一节](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Obfuscation-and-Mixins)中为什么如此详细地定义了我们所谓的 *[混淆边界](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Obfuscation-and-Mixins#2-resolving-the-identity-crisis---defining-the-obfuscation-boundary)*。你可能还记得那部分：


> 任何Mixin特定机制……**将始终使用某种注解进行修饰**。这使得[Mixin注解处理器（Annotation Processor）](https://github.com/SpongePowered/Mixin/wiki/Using-the-Mixin-Annotation-Processor) 能够找到它们，并将处理它们的混淆。

……事实上就是这样。

如果你仔细想想，这是完全合乎逻辑的：重写方法不会“知道”它将重写一个目标方法*直到应用Mixin*，此时，它发现它的目标点已经存在原始方法，并且达成了它的重写任务。当不涉及混淆时，这是可行的，但是当存在混淆时，这是非常痛苦的，因为我们需要一些方法“链接”重写与所需的目标方法，以便让注解处理器生成混淆表条目。

为了做到这一点，我们使用一个简单的注解`@Overwrite`。

```java
    /**
     * 添加注解到重写中，将其与将被混淆的目标方法链接
     */
    @Overwrite
    public int getLevel() {
        return Math.max(this.level, 0);
    }
```

##### 1.3.1 使用 `@Overwrite` 注解

用`@Overwrite`来修饰一个方法会使注解处理器在编译时寻找目标方法。如果没有找到混淆映射，注解处理器将发出一个错误。

这意味着：

* 在你的Mixin中为一个不是混淆的方法定义一个**重写**，你应该简单地**声明方法**。

* 为一个混淆方法定义一个**重写**，你应该**声明该方法并用`@Overwrite`注解注解它**。

你还应该记住，任何包含重写方法的Mixin*不能超出目标类的范围*。原因是，即使所有目标类在MCP环境中定义具有相同名称和相同签名的方法，但这并不表示所有混淆环境的每个方法都具有唯一名称。可能可以使用别名来解决此问题，但这是不推荐的。

#### 1.4 `@Overwrite` 注解的其他用法

`@Overwrite`注解还有一个最终用途。为目标类中的`public static`方法定义重写。

默认情况下，在Mixin中声明一个`public static`方法会引起`Id10t Error`，因为无法调用以这种方式定义的方法！然而应该知道的是，重写为这种无意义的操作提供了一个用例。用`@Overwrite`注解方法将绕过限制，并允许在Mixin中定义`public static`方法。

### 2. 内部代理方法

虽然常规的重写的行为易于理解和相当可预测，但是它们缺乏灵活性，给Mixin设计来带了一些挑战，而这些挑战不容易克服。好消息是Mixin提供了内置功能来应对这些问题，坏消息是这个功能最初看起来会相当复杂。为了更清楚地理解这些功能，我们将逐步解决这个问题。

#### 2.1 内部的嘎嘎叫和寻找混淆的鸭子类型

Mixin的关键任务之一是通过使用Mixin将自己的接口应用到现有对象，在Java应用程序中提供伪鸭子类型功能。有时现有类中的方法*已经*实现了接口方法，我们称这种类型的方法为*内部（Intrinsic）*方法，因为它是现有类的一部分：实际上我们的类已经——从本质上——就知道如何嘎嘎叫。

但是有一个问题：如果内部方法被混淆了会发生什么？答案是——对象一旦被混淆，就不再会嘎嘎叫，一旦越过混淆边界，接口约定就会被破坏。

让我们来看一个简单的例子。在本例中，我们将使用与前一篇文章中相同的类和接口，但我们将假设`Identifyable`接口不会与目标对象冲突：

![](mixin_tut_24.png)

正如我们所见的，Mixin中不需要实现`getID`，因为类`Foo`本质上已经实现了接口。但是，混淆之后，类、字段和方法名称都发生了变化，我们遇到了一个问题：

![](mixin_tut_25.png)

既然方法不再有实现，任何使用者如果试图调用该方法，都会引发`AbstractMethodError`。

有一个使用我们已知功能的方法可以解决这个问题：

* __用原始方法的副本重写该方法__
 这似乎是最直白的，当然也是最简单的方法。如我们所知，省略`@Overwrite`注解会导致重写*不*被混淆。这意味着，在我们的开发环境中（方法和字段名不被混淆），该方法将简单地重写目标中的现有方法：

 ![](mixin_tut_26.png)
 
 在混淆之后，重写神奇地转换为*新*的访问器，因为方法合并的语义意味着该方法将简单地添加到目标类：
 
 ![](mixin_tut_27.png)
 
这种方法的两个主要缺点是，首先，它需要将原始方法复制到Mixin中，这对于简单的访问器来说是可以接受的，但对于更复杂的方法可能是有问题的，因为它再次使我们处于需要手动保持功能与目标方法等同的处境。如果目标方法改变，那么我们必须更新重写。其次，我们可能最终需要为那些我们并不真正感兴趣的字段创建影子，并且宁愿通过现有类的公共约定（例如，通过原始访问器）进行访问，而对于更复杂的方法，这可能有更多的影子字段。除了复制原始方法的功能之外，我们对这些字段并不真正感兴趣，因此添加它们只会造成代码混乱。

**内部代理**方法允许我们在出现这种情况时对重写过程进行细粒度控制（Finer-grained Control）。

#### 2.2 优雅的重写不会为权利受损而斗争

我们可以通过引入一种新的重写方式来改善这种状况，具体来说：

* `“不要重写方法如果它已经存在（是内部的）”`

在此情况下，我们创建了上述重写，但是用`@Intrinsic`注解来修饰它。这有效地声明了重写是针对内部方法的，如果找到了方法，则不应该进行重写。

虽然这不是一个巨大的改进，但从所有外部代码（目标代码库中的代码）的视角来看，它确实意味着方法的**原始约定保证被保留*，这减少了底层方法可能发生的更改，并且不必担忧发生的更改在我们的重写中反映出来。通过运行原始方法始终存在，并且只在生产中添加我们的新方法（供我们自己的代码使用），我们有机会能稍微提高些稳定性。

#### 2.3 代理内部方法

当然，这仍然意味着，调用我们的鸭子类型接口的*代码*调用方法，可能最终与基于环境的方法的不同实现交互，这取决于方法的性质。我们真正希望能够做的是调用原始方法，并在它周围封装一些我们自己的逻辑。

幸运的是，我们可以把我们的重写定义为一个**内部代理**。

**内部代理方法（Intrinsic Proxy Method）**通过改变重写的行为以工作，使得*原始*方法是被*移动*而不是重写。然后，我们可以在所有情况下从重写中调用*原始*方法。然而，为了定义新的代理方法，我们需要`@Shadow`原始方法——这会产生冲突。幸运的是，我们确切地知道如何处理冲突：我们使用**软实现**！

让我们在确切的方向上迈出一小步，将新的访问器转换为软实现：

```java
@Mixin(Foo.class)
@Implements(@Interface(iface = Indentifyable.class, prefix = "id$"))
public abstract class MixinFoo {

    @Shadow
    public abstract int getID();

    /**
     * 该方法将成为我们的内部代理方法，
     * 它调用访问器的原始（影射）版本。
     */
    public int id$getID() {
        // 调用原访问器
        return this.getID();
    }
}
```

当然，在应用Mixin时，前缀将被删除，这意味着在Mixin合并之后，我们将以冲突结束。正如我们所知，Mixin会把这个冲突当作一个*重写*，让我们回到起点！

![](mixin_tut_28.png)

然而，情况变得更糟，因为新方法中对`this.getID()`的调用**现在变成了自引用**，如果调用该方法，将导致堆栈溢出，因为它将递归地调用自己，直到JVM用完堆栈空间！

这就是我们的新朋友`@Intrinsic`再次拯救的地方。`@Intrinsic`注解具有间接行为，如果目标内部方法已经存在，则它允许目标内部方法不被替换，而是被移动。

```java
    /**
     * 该方法将成为我们的内部代理方法，
     * 它调用访问器的原始（影射）版本。
     * 当方法被重写时，
     * 使用displace参数以避免重入。
     */
    @Intrinsic(displace = true)
    public int id$getID() {
        // 调用原始访问器
        return this.getID();
    }
```

添加`displace`参数会导致内部重写以以下方式表现：

* 如果内部对应项不存在（例如，如果方法在具有不同名称的混淆环境中），则新代理方法将像往常一样简单地添加到目标类中。
* 如果内部对应项确实存在，那么会发生三件事：
 1. 将内部项重命名为新名称。
 2. 代理中对内部项的引用将被更新为新名称。
 3. 然后将代理方法像以前一样添加到目标类中。

这种新的方式使我们自食其力，因为我们确保自己的代码总是调用代理方法，而且原始访问器的约定也总是保持不变。我们还不需要为任何不相关的目标类成员添加影子，而只需要为内部对应项添加影子，从而使得Mixin代码更加清晰。

我们的新Mixin行为图是这样的：

![](mixin_tut_29.png)

### 3. 约束

如上所述，为了避免破坏目标程序，需要仔细地使用重写功能。虽然在Mixin代码库中对重写使用严格的管理方案会有很大的帮助，但是在现实中管理你的产品可能更棘手，特别是当用户在意外环境中部署你的产品时，例如，使用你打算混合的软件的较新版本。

因此，约束提供了此前没有的健全检查，前提是你能够提供相关信息到可能需要约束的环境。

#### 3.1 在应用程序环境中管理约束

约束采用与单个整数值相关联的字符串标记的形式。这些标记的值必须通过*标记提供器（Token Provider）*的实例与`MixinEnvironment`注册到Mixin环境中。

标记完全取决于你，但一般来说，你将希望表示目标应用程序环境的某些方面。假设你能够从应用程序的单例实例中获取目标应用程序的构建号：一个简单的标记提供程序可能如下所示：

```java
public class MyTokenProvider implements ITokenProvider {
    public Integer getToken(String token) {
        if ("BUILD".equals(token)) {
            return TargetApplication.getInstance().getBuildNumber();
        }
        return null;
    }
}
```

该标记提供器为标记`BUILD`返回了应用程序构建号。它为所有其他标记返回`null`以表示提供器不支持这个标记。在[引导Mixin库](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment)
时，我们必须注册标记提供器的实例。

#### 3.2 使用约束

一旦我们在环境中定义了标记，我们就可以定义对重写的约束：

```java
@Overwrite(constraints = "BUILD(1234)")
public void someHackyOverwrite(int x, int y) {
    // 做一些修改
}
```

该修改方法用一个约束来定义，该约束指示标记`BUILD`**必须被定义**，并且**其值必须为1234**。如果不满足此约束，则Mixin处理器将发生错误并使应用程序崩溃。

我们也可以定义在一定区间内定义约束，定义一系列可以写入的值：

```java
@Overwrite(constraints = "BUILD(1230-1240)")
```

这允许在1230到1240（包括）之间的值通过，并且也可以写成这样：

```java
@Overwrite(constraints = "BUILD(1230+10)")
```
下面给出了约束标识符的完整列表：
<table>
    <thead>
        <tr>
            <th>约束字符串</th>
            <th>意义</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td valign="top"><tt>()</tt></td>
            <td valign="top">该标记必须存在于环境中，但可以有任何值</td>
        </tr>
        <tr>
            <td valign="top"><tt>(1234)</tt>
            <td valign="top">标记值必须为<tt>1234</tt></td>
        </tr>
        <tr>
            <td valign="top"><tt>(1234+)</tt><br />
<tt>(1234-)</tt><br />
<tt>(1234&gt;)</tt></td>
            <td valign="top">所有这些变体都有相同的意义，都可以认为是“1234及以上”</td>
        </tr>
        <tr>
            <td valign="top"><tt>(&lt;1234)</tt></td>
            <td valign="top">小于1234</td>
        </tr>
        <tr>
            <td valign="top"><tt>(&lt;=1234)</tt></td>
            <td valign="top">小于或等于1234（相当于<tt>1234&lt;</tt>）</td>
        </tr>
        <tr>
            <td valign="top"><tt>(&gt;1234)</tt></td>
            <td valign="top">大于1234</td>
        </tr>
        <tr>
            <td valign="top"><tt>(&gt;=1234)</tt></td>
            <td valign="top">大于或等于1234（相当于<tt>1234&gt;</tt>）</td>
         </tr>
        <tr>
            <td valign="top"><tt>(1234-1300)</tt></td>
            <td valign="top">值必须在1234至1300（包括）之间</td>
         </tr>
        <tr>
            <td valign="top"><tt>(1234+10)</tt>
            <td valign="top">值必须在1234至1234+10之间（1234-1244包括）</td>
        </tr>
    </tbody>
</table>

可以被表示为整数的目标环境的任何特征都可以作为约束的基础。例如，布尔状态可以表示为1或0。

#### 3.3 选择约束

究竟使用哪种约束取决于环境，以及预计方法的变动性。例如，重写一个简单的访问器可认为比重写一个复杂的访问器风险要小得多，因此可以得到更大的约束。对于极易变的方法（在本例中，变动性是方法改变的可能性），那么较小的约束可能是个好主意。

当然，确切值的“大小”将取决于标记，例如，在一个每天有多个构建的项目上使用构建号，“小”的值可能是100。而对于一个每年只变化几次的项目来说，“小”的值可能是2。当在环境中定义标记时使用你的判断，并在应用程序的开发人员说明中说明标记的预期变动。

### 4. 总结

重写和内部代理提供了强大的功能，但是必须仔细设计它们，同时考虑诸多因素，并应尽可能避免使用重写和内部代理。

使用重写作为代码库中的另一个工具，可以在设计Mixin时提供很大的灵活性和权利，不计后果地使用它们，并且不考虑潜在的陷阱，这几乎肯定会在你的应用程序生命周期之后导致问题。
