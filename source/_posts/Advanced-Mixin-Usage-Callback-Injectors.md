---
title: 高级Mixin用法——回调注入器
date: 2018-12-05 12:51:16
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Advanced Mixin Usage Callback Injectors](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors)

如果你还没有读过**介绍系列**的话，现在回头吧，因为*这里是龙潭虎穴*……

还不走？好吧，我们开始吧。

### 1. 从以太诞生的代码——了解注入器

如果你已经读过了介绍系列，你就知道如何使用Mixin处理器将你提供的代码合并到**目标类**中。实际上，即使有了如*内部代理*那样更复杂的功能，最终得到的目标类的代码也只是原始字节码和你提供的新代码的混合。

**回调注入器（Callback Injector）**是你将学会的第一个不同以往的Mixin技术：有了**回调注入器**，可以让Mixin处理器为我们**生成新代码**。

**回调注入器**由两部分组成：

* 一个**常规Mixin方法**即一个**处理方法（Handler Method）**，该方法通过含有一些自定义代码的Mixin添加到目标类中。
* 一个**注入点（Injection Point）**定义 - 这是一个注解以告诉Mixin处理器在何处创建在**现有方法**（或称为**目标方法**）内的指令，该方法将**调用我们的处理方法**。

在我们探讨更多细节之前，来介绍一下我们的学习样例。下图展示了一个非常简单的类`Flard`中的一些代码：

![](flard_00.png)

`Flard`类包含：

* 一个`protected`字段`position`
* 一个`public`方法`setPos`
* 一个`protected`方法`update`

`Flard`的内部行为是每当属性发生改变时，属性的设置器总是在返回之前调用`update()`方法。`update`方法在属性改变时将对象状态发送到服务器。

> 注意，即使该方法返回`void`，我仍然在代码中囊括了一个显式的`return`。原因是——在字节码中——没有所谓的*隐式*返回，在方法的字节码中总是有一个`RETURN`操作符。在我们之后考虑*注入点*时，这是一个重要的事情。

让我们看看从一些外部代码调用`setPos`设置器的执行流程：

![](flard_01.png)

这里并没有发生什么特别的事情，箭头精确地指示了我们期望给定代码发生的事情。

#### 1.1 概括——你已经知道的东西

到目前为止很无聊。假设我们想进入`update()`操作，我们想要添加一些自定义逻辑——也许通知一些观察者类——在调用`update()`时。我们已知可以通过用[修改后的版本重写它](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Overwriting-Methods)甚至使用 *[内部代理](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Overwriting-Methods#2-intrinsic-proxy-methods)替代*以保持原始方法。然而，我们也知道重写在可维护性和互操作性方法有**很多**[缺点](https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---Overwriting-Methods#12-with-great-power-comes-great-possibility-to-screw-things-up)。

那么，如何在不重写的情况下向方法添加自定义逻辑呢？

答案是：*告诉方法调用我们的自定义代码！*

![](invoke.png)

#### 1.2 你要调用谁？onUpdate！

在我们了解注入器的旅程中，第一步实际上也是我们已知的：将*新*的方法添加到目标类中。我们将注入的方法称为**处理方法**，并且通常我们使用前缀“`on`”标记处理方法，这与许多框架中使用的事件接受器的命名约定非常相似。

我们将本例中这个新的**处理**方法命名为`onUpdate`，因为我们将挂钩到`update`方法。

> 使用紧随目标方法名称的`on`术语有助于Mixin代码的可读性，因为任何阅读Mixin代码的人都可以在不必检查注入器定义的情况下了解该方法所含的内容。

因此，假设我们的第一步是使用Mixin将新的**处理**方法`onUpdate`添加到目标类中。在本例中，**处理**方法简单地对某个假想名为`Observer`的单例观察者对象调用`foo`：

![](flard_02.png)

好，一切都很好，但是我们的新方法基本上只是呆在那里，除此之外什么都不做，所以让我们看看我们如何做注入器的实际*注入*部分：

#### 1.3 头（HEAD）, 肩膀, 膝盖, 返回（RETURN）, 膝盖, 返回（RETURN）!

在开始之前，我们首先要做的是识别目标方法的各部分。让我们用记号标记`update()`方法，这些记号标明了我们能够容易识别的一些区域：

![](flard_03.png)

记号标明了该方法的各部分剖析：

* **HEAD**表示方法体中的*第一个操作符之前*的点，即方法的“头”。
* **RETURN**表示方法中的*`RETURN`操作符之前*的点。

在一个方法中，我们还可以标记出其他要点，但是现在我们仅重点关注这些简单的标记，在注入器术语中，这些方法中的位置被称为**注入点**。

#### 1.4 嘿，我只是混在一起，这确实疯狂，但我是一个方法，所以也许该调用我？

现在我们准备链接这些点。首先我们将用注入器声明来修饰我们的`onUpdate`处理方法：

![](flard_04.png)

如你所见，`@Inject`注解需要两条信息：要注入的**方法**和要注入的**点**。在代码中看起来是这样的：

```java

@Inject(method = "update", at = @At("HEAD"))
protected void onUpdate() {
    Observer.instance.foo(this);
}
```

当Mixin处理器应用Mixin方法时，它看到`@Inject`注解并在**目标**方法中生成调用**处理**方法的代码：

![](flard_05.png)

我们的**处理**函数的注入回调以紫色显示。与重写不同，**目标**方法的原始行为是不受影响的，我们的**处理**方法只是在方法开始时接收回调。

最终，使用注入器的步骤总结如下：

1. 识别**注入点**（接下来有更多关于此的介绍）。
2. 编写你的**处理方法**。
3. 用一个`@Inject`注解来修饰处理方法以创建注入关系。

现在我们理解了基础知识，我们已经准备好深入研究这个问题。

### 2. 构建处理方法

为了引入注入器的概念而简化了上述介绍。实际上，所有注入器的**处理方法**必须在其参数中接收`CallbackInfo`对象。该对象由回调生成，并作为句柄，在编写*可取消注入器*时可使用该句柄（更多信息见下文）。

我们此前的示例代码实际上是这样的：
```java

@Inject(method = "update", at = @At("HEAD"))
protected void onUpdate(CallbackInfo ci) {
    Observer.instance.foo(this);
}
```

> 上面的示例也得到了简化，因为目标方法不接收参数并返回`void`。

#### 2.1 抢占目标参数

当注入到有参数的**目标方法**时，**目标方法**的参数也可以作为注入的一部分传递给**处理方法**。让我们看到上例中的`setPos(int, int)`方法：

```java
/**
 * 一个目标方法，setPos
 */
public void setPos(int x, int y) {
    Point p = new Point(x, y);
    this.position = p;
    this.update();
}
```

如果我们要为这个方法创建一个注入器，我们可以选择在我们的**处理**方法中使用两个整型参数（`x`和`y`）。我们可以简单地将它们添加到我们的**处理方法**的签名中：

```java
/**
 * 处理方法，onSetPos，注意两个int变量x和y出现在CallbackInfo之前
 */
@Inject(method = "setPos", at = @At("HEAD"))
protected void onSetPos(int x, int y, CallbackInfo ci) {
    System.out.printf("Position is being set to (%d, %d)\n", x, y);
}
```

注意我们仍需包含`CallbackInfo`参数。注入`setPos`的代码先现在看起来是这样的，注意调用在调用**处理**方法`onSetPos`时传递了参数`x`和`y`：

![](flard_06.png)

方法参数和CallbackInfo一同为注入提供**环境（Context）**，并允许你相应地更改处理器的行为。例如：

```java
/**
 * 处理方法，onSetPos
 */
@Inject(method = "setPos", at = @At("HEAD"))
protected void onSetPos(int x, int y, CallbackInfo ci) {
    // 检查如果设置位置到原点
    if (x == 0 && y == 0) {
        this.handleOriginPosition();
    }
}
```

### 3. 可取消注入

到目前为止，我们的注入根本没有改变目标方法的结构，它只是调用我们的处理函数，使它执行我们需要的任何操作，而不改变目标方法。**可取消注入**允许我们创建可以提前从目标方法*return*的注入。

从上面的`setPos`示例将注入的代码可以看出，注入的代码包括创建一个新的`CallbackInfo`实例，然后调用**处理方法**。但是，现在我们将向`@Inject`注解中的注入器声明添加`cancellable`标识。我们还将为处理方法体添加一些逻辑：

```java
/**
 * 可取消注入，注意我们在注入器注解中将"cancellable"标识设置为"true"
 */
@Inject(method = "setPos", at = @At("HEAD"), cancellable = true)
protected void onSetPos(int x, int y, CallbackInfo ci) {
    // 检查位置是否设置到原点，并执行自定义逻辑
    if (x == 0 && y == 0) {
        // 一些自定义逻辑
        this.position = Point.ORIGIN;
        this.handleOriginPosition();
        
        // 就像原方法一样调用update()
        this.update();
        
        // 将回调标记为已取消
        ci.cancel();
    }
    
    // 此时正常执行，不执行自定义处理
}
```

在上述代码中，**处理方法**检查位置是否设置为`(0, 0)`，并执行一些自定义逻辑，将回调标记为**已取消**，以使目标方法（`setPos`）在**注入之后立刻返回**。我们可以看到目标方法是如何处理注入到`setPos`的代码，看起来就像这样：

![](flard_07.png)

如你所见，将注入标记为可取消的，会导致Mixin处理器注入代码时会检查`CallbackInfo`是否被标记为已取消，如果是，则立刻返回。

> 注意：在*不可取消*的`CallbackInfo`上调用`cancel()`将引发异常！

对于我们**在方法`HEAD`处，并且有条件地返回的注入**，被称为**短路注入（Short Circuit Injection）**，这对于那些本来是需要`@Overwrite`的东西是非常好的替代方案。也可以写一个总是取消的短路注入，这种类型的注入器被称为**永久短路注入（Permanent Short Circuit）**或**阻断注入（RoadBlock Injection）**。

> 注意，大多数情况下，阻断注入最好是重写。其原因是因为它允许其他转换器对原始方法进行操作，而不会引起错误；还因为将注入的代码保持在单独的方法中，这使得在注入的代码中发生异常时，堆栈追踪更有用。

### 4. 瞄准——注入点

此前，我们已经遇到了两个不同的**注入点**：`HEAD`和`RETURN`，并且看到了第一个示例。在定义所有类型的注入器时，理解注入点是至关重要的，因为你选择的注入点类型将取决于你希望注入达成什么目的。

`HEAD`和`RETURN`注入点是特殊的，因为它们是*保证成功的*唯一注入点，这是因为在一个方法中*总是*至少有一个RETURN操作符，并且总会自然有一个方法“头”。

#### 4.1 在草垛中寻找操作符

关于注入点，首先需要明白的是，它们本质上是*查询*目标方法运行字节码中一个*或更多*个与其标准匹配的操作符。没错：一个注入点可以匹配不止一次。

举一个例子，让我们思考一下`RETURN`注入点的含义。`RETURN`注入点的定义如下：

* `RETURN`匹配目标方法中的*所有RETURN操作符*。

Mixin处理器将始终在匹配的操作符*之前立刻*注入回调，因此使用`RETURN`将始终在方法返回之前立刻注入回调。

让我们看一个例子：

![](flard_08.png)

在本例中，我们修改了`setPos`方法，并在方法中添加了一些带有显式返回值的额外逻辑。除了方法结尾的*隐式*返回以外，这还意味着现在方法体中含有*两个*RETURN操作符。当我们标记方法中的注入点时，我们可以看到这映射在`RETURN`所标识的注入点中：

![](flard_09.png)

为了区分所标记的操作符，由注入点标识的每个操作符都以从零开始的数字标记，该操作符索引被称为**序号（Ordinal）**，并且是从零开始的索引。

假设我们想要编写一个只在第一个RETURN操作符上放置的回调注入器：

```java
@Inject(method = "setPos", at = @At(value = "RETURN", ordinal = 0))
protected void onResetPos(int x, int y, CallbackInfo ci) {
    // 处理方法的逻辑
}
```

在`@At`中指定`ordinal`值意味着我们只在*第一个*RETURN操作符之前进行注入。

> **注意:**你可能会问自己：*这个示例注入与仅注入`reset()`方法有什么区别？它会在同一个点被调用，对吧？*
> 
> 选择正确的注入器目标可能是极其主观的，并且很多时候将取决于你试图通过特定注入实现什么，以及其他因素，例如所讨论的对象的子类，以及范围中可用的变量。
> 
> 例如，在上述发方法中，注入`reset()`会在同一点引发回调，但是如果`reset()`被子类重载了怎么办？`reset()`方法也不接收参数`x`和`y`的副本，这是我们可能需要的。另外，如果从代码中的其他点调用`reset()`方法会怎么样呢？所有这些事情都应该考虑。
> 
> 选择合适的注入点在很大程度上取决于所讨论的类的结构（和层次结构）以及注入后打算做什么。在确定注入点是，应该考虑注入目标的使用及其性质的所有方面。

#### 4.2 注入点的性质

在进一步讨论之前，需要了解一些关于注入点的关键内容：

* 除少数情况外，注入器将把其注入的代码置于**注入点标识的操作符之前**。这应该是很直观的，一些例子如下：
 * `RETURN`标识方法中的`RETURN`操作符，注入发生在方法返回之前。
 * `HEAD`标识方法中的*第一个*操作符，注入发生在方法的开始处。
 * `INVOKE`（参见下文）标识方法调用，注入发生在方法调用之前。

* 由于它们是高效的查询，注入点**可能不会返回结果**。例如，假设在上述的示例中，我们指定`RETURN`作为查询，其`ordinal`值为`2`。由于在该方法中只有两个返回操作符，所以注入点将不匹配操作符。你可以使用约束指定可接受的上限和下限。

* 虽然对于一个**给定的方法**注入点是**确定的**，但是当注入**一个变化的代码库**时，这并不能消除维护负担。除了`HEAD`（它总是匹配相同的点），当目标代码库改变时，应该检查所有的注入点，尤其是那些使用`ordinal`偏移量或具有其他参数化设置的注入点，以确保它们始终正确。

* 定义更复杂的注入点（下面将详细介绍）是Mixin中少数几个必须*弄脏你的手*并查看**目标方法**的字节码的地方之一。这通常是必要的，所以你可以选择最适合你的注入操作符，一个好的反汇编器将在此为你提供巨大的帮助。

#### 4.3 其他类型的注入点

你可用的注入点中，除了`HEAD`和`RETURN`以外，还可以选择使用其他预定义的注入点：

<table width="100%">
    <tr>
        <th>注入点</th>
        <th>代码识别方式</th>
    </tr>
    <tr>
        <td><tt>INVOKE</tt></td>
        <td>查找方法调用并在其之前注入</td>
    </tr>
    <tr>
        <td><tt>FIELD</tt></td>
        <td>查找字段读写并在其之前注入</td>
    </tr>
    <tr>
        <td><tt>NEW</tt></td>
        <td>查找<tt>NEW</tt>操作符（对象创建）并在其之前注入</td>
    </tr>
    <tr>
        <td><tt>JUMP</tt></td>
        <td>查找跳转操作符（任何类型）并在其之前注入</td>
    </tr>
    <tr>
        <td><tt>INVOKE_STRING</tt></td>
        <td>查找方法调用，该方法接受单个<tt>String</tt>并返回<tt>void</tt>，或者说该方法接受常量字符串作为参数。这可以主要用于查找对<tt>Profiler.startSection(nameOfSection)</tt>的调用。</td>
    </tr>
    <tr>
        <td><tt>INVOKE_ASSIGN</tt></td>
        <td>查找方法调用，该方法调用后返回一个值，并在该值被分配给本地变量之后立刻注入。<b>注：这是唯一一个在注入点<em>之后</em>的注入</b></td>
    </tr>
</table>

更多细节请看[注入点参考](https://github.com/SpongePowered/Mixin/wiki/Injection-Point-Reference)

### 5. 具有非Void返回类型的目标方法

到目前为止，我们仅考虑注入一个返回`void`的目标方法。当注入具有非`void`返回类型的防辐射，注入器**处理方法**将替代性地接收一个`CallbackInfoReturnable`。`CallbackInfoReturnable`不同于它的父类`CallbackInfo`：

* `CallbackInfoReturnable`是泛型的，方法的返回类型是它的类型参数。
* 当取消一个具有返回类型的`cancellable`回调时，可以通过调用`setReturnValue`而不是`cancel`来指定从该方法返回的值。

正如你所想的，当注入Getter，或当在`HEAD`注入使一个方法**短路**时，这是非常有用的。让我们来看一个返回值的示例方法：

![](flard_10.png)

`getPos`方法是一个典型的*Getter*方法，因为它的方法体仅返回受保护的字段的值。让我们添加一个注入`HEAD`的Mixin，如果字段为`null`，则返回默认值：

![](flard_11.png)

注入器首先将我们的**处理方法**合并到目标类中，然后将代码注入到**目标方法**中以处理取消。

![](flard_12.png)

注意注入的代码与此前展示的`void`类型的短路代码的区别。这次如果取消回调，注入器返回我们在`CallbackInfoReturnable`中设置的值。我们的Mixin代码如下所示：

```java
@Inject(method = "getPos", at = @At("HEAD"), cancellable = true)
protected void onGetPos(CallbackInfoReturnable<Point> cir) {
    if (this.position == null) {
        // setReturnValue 隐式 cancel() 回调
        cir.setReturnValue(Point.ORIGIN);
    }
    
    // 注意，如果处理方法正常返回，则方法继续正常运行，就像可取消注入不调用cancel()的正常情况一样。
}
```

可以想象，这种注入器非常强大，并且不限于注入`HEAD`，你可以使用任何你希望的注入点。然而，带使用可取消可返回注入时，会发生一些特殊情况：

你可能会想 *“但在这个非常简单的方法中，`HEAD`和`RETURN`的基本上意思是一样的，对吧？”*

可以原谅你这样想，因为这似乎是逻辑推断的结果，然而在实际中，单个语句`return this.position`实际上包含两个操作：

1. 获取字段`position`的值。
2. 返回这个值。

这个值临时储存在方法的*参数堆栈*中，你可以将这些视为临时的、不可见的变量，这些变量在JVM操作值时储存这些值。从实际的角度来看，这意味着`HEAD`和`RETURN`在方法中实际上是位置分开的！

![](flard_13.png)

由于我们知道，`RETURN`操作符必须返回一个值，因此我们知道，无论何时在`RETURN`注入，该值都可用于访问。注入处理器处理这种情况，并将返回值“捕获”并传递给处理方法的CallbackInfo中：

![](flard_14.png)

这种方法有很多好处：

* 现在我们的注入器Key更松散地耦合到**目标方法**的实现中。换句话说，如果**目标**方法更改，我们的处理方法不需要知道更改，因为它只关心返回的值。
* 如果**目标**反复有多个`RETURN`操作符，则返回的值仍可被处理，而不需要额外的上下文。
* 它运行我们进行“观察者”注入，该注入仅*检查*返回的值，而不实际更改它或担心方法的实现。

例如，让我们更改上述例子，用`RETURN`代替`HEAD`。我们关系的是让这个方法不返回`null`，所以我们的代码变成：

```java
@Inject(method = "getPos", at = @At("RETURN"), cancellable = true)
protected void onGetPos(CallbackInfoReturnable<Point> cir) {
    // 检查捕获的返回值
    if (cir.getReturnValue() == null) {
        // 如果为空，设置默认值为返回值
        cir.setReturnValue(Point.ORIGIN);
    }
}
```

### 6. 回调注入器特性

#### 6.1 多注入点

应该注意的是，`@Inject`注解的`at`参数实际上是数组类型。这意味着可为单个注入器处理方法指定多个注入点。

#### 6.2 目标通配符

当为注入器指定目标`method`时，添加星号（`*`）的方法名称可指定Mixin处理器匹配所有具有指定名称的方法，不管它们的签名如何。这允许单个注入器以多个方法为目标。

使用这种语法会使捕获方法参数变得不可能，因为方法参数会因方法而异。然而，它使观察类型的注入变得简单。例如，思考下面的注入：

```java
@Inject(method = "<init>*", at = @At("RETURN"))
private void onConstructed(CallbackInfo ci) {
    // 初始化资源
}
```

这个注入器向类中的所有构造方法注入回调。如果目标类具有多个重载的构造方法，并且你仅希望将其注入到所有构造方法中，那么这非常有用。

目标通配符可以与任何方法名一起使用，但是，如果通配符匹配具有`void`返回类型和非`void`返回类型的方法，则注入将失败并出错，因为对于非`void`的目标需要使用`CallbackInfoReturnable`。

### 7. 回调注入器的思考与局限性

#### 7.1 注入构造方法

现在Java代码中的构造方法相当简单，并限定了三个简单的约束：

1. 在任何其他代码之前都必须调用`super`。
2. 必须初始化任何`final`字段。
3. 作为第1点造成的结果，你不能在`super`调用中内联调用实例方法，这里进行的任何调用都必须是静态的。

但在*字节码*层面，构造方法更加精致。由于编译后的`<init>`方法混杂着原先构造方法的代码，任何类的字段初始化器（复制到所有构造方法），以及在某些情况下合成的（编译器生成的）代码（例如`Enum`的构造方法）。由于它们的性质，它们就是字节码层面变换的雷区。

因此，Mixin对注入器施加以下限制：

* **为构造方法支持的唯一注入点是`RETURN`注入点**。之所以强加此限制，是因为在调用处理方法代码之前，没有其他明智的方法可以确保类已经完全初始化。

如果要注入构造方法，**必须**指定`RETURN`为注入点。

#### 7.2 故障状态

像其他Mixin功能一样，回调注入器被设计为*快速失效*和*故障安全*的。这通常意味着如果注入器发生故障，它通常会做两件事之一：

1. “静默”故障（除非启用了Mixin调试）并保持**目标方法**不变（但**处理**方法仍将合并到目标类中）。
2. 使用确定性的错误消息故障：例如，当注入器找到其目标操作符，但**处理**方法有着不正确的签名时。

注入器的设计使得任何故障都**不会**破坏方法字节码，它要么成功，要么绝对失败。

静默故障通常发生在注入点不匹配操作符的注入器上。当目标方法被更改或替换时可能会发生这种情况，并且可能非常有用，因为可以创建“重载”注入器来适应不同的情况。

然而，有时候注入器的成功是很重要的。也许应用程序的稳定取决于它，或者失败不是预期的状态，应用程序应该被关闭。无论那种方式，有时必须能够*确保*注入器成功（如果不成功，则引发故障状态）。使用`expect`和`require`参数是可能做到这一点的。

##### 7.2.1 Require

`require`参数很简单，为`require`指定特定值要求注入器必须成功*至少几次*。

```java
@Inject(method = "foo", at = @At(value = "INVOKE", target = "someMethod"), require = 2)
private void onInvokeSomeMethodInFoo(CallbackInfo ci) {
    // ...
```

在本例中，我们希望我们的注入点匹配**目标**方法中的2个调用.指定`require`意味着如果少于2次注入结果，就会发生错误。

还可以在Mixin配置文件中为`require`指定一个配置域的值。

##### 7.2.2 Expect

有时在运行时故障是容许的，但是当开发Mixin时，你希望能够使用`require`功能（当注入失败时出错）而不必到处撒`require`，并且不需要记住在代码投入生产之前删除他！

`expect`参数允许你精确地执行。`expect`的工作方式与`require`完全一样，除了它仅在将`mixin.debug.countInjections`系统属性设置为`true`是才对其进行处理。这允许你在开发检查注入器是否工作，并使它们在生产中快速失效。

##### 7.2.3 全局注入器设置

你可以在你的Mixin配置文件中为`require`指定一个配置域的默认值，其包含在一个`injectors`块中，你可以覆盖配置中的所有Mixin的`require`默认值：

```json
{
   "injectors": {
        "defaultRequire": 1
    }
}
```

##### 7.2.4 注入器组

虽然`require`语句运行你规定单个注入器的行为额，但你可能会遇到希望为单个情境提供多个替代注入器的情况。如果Mixin面向多个环境，或者已知其他转换器已定义良好的方式修改特定目标方法，则这非常有用。

在这些情况下，你可以提供两个或多个注入器，其中对于给定的环境，只有一个预期成功。这当然提出了一个问题，即如何利用这些注入器的`require`值，因为如果在给定情景中预计至少有一个注入器会故障，那么这就不可能使用`require`。

为了解决这一问题，注入器支持*组*的声明。使用注入器组允许*为组*指定`min`和`max`注入次数，聪哥确保注入仍然可以验证为有效，但是应该只发生指定的注入次数。

为了使用注入器组，只需让每个注入器**处理方法**声明`@Group`注解。第一个注解应该指定`min`和（可选地）`max`。如果在单个Mixin中使用多个注入器组，那么应该在Mixin中的所有`@Group`注解上指定该组的唯一`name`。

#### 7.3 重写处理方法的行为

当定义回调注入器**处理**方法时，将使用正常的Mixin行为将方法合并到**目标类**中。然而，这对子类，特别是派生类型中定义的注入器有影响。

* **处理方法在合并前重命名**——所有**处理**方法再被合并到目标类之前被*修饰*。这确保了如果*相同*类的另一个Mixin定义了*相同的*注入器，那么这两个处理方法将不会冲突。

* **处理方法使用适合其访问级别的操作符来调用**——如果你的处理方法是`private`，那么将使用`INVOKESPECIAL`（静态绑定）调用它，如果**处理**方法是非私有的，则将使用`INVOKEVIRTUAL`调用它，这将允许你在派生的Mixin中`@Override`处理方法。

* **如果在派生的Mixin中`@Override`一个处理方法，它将被重命名以匹配其超Mixin对应方法的修饰**——这样做是为了使得派生类中的方法永远不能“意外”覆盖你的**处理**方法。

通常，除非明确计划为特定处理方法使用重写语句，否则建议**处理**方法是`private`的。

### 8. 总结

这似乎要思考很多，回调注入器功能强大，非常精细，因此这不是一种合理的思考方式！让我们回顾一下要点，把一切都理清楚：

* 回调注入器只是常规的Mixin方法，具有*注入*回调到目标类中一些位置的特殊行为。

* 它们总是由目标内部的**处理方法**、**目标方法**和一些*注入点*组成。

* 回调注入器方法总是接收一个`CallbackInfo`，并且可以接收其他参数，例如**目标**方法的参数。

* 回调可以是*可取消的*，允许从目标方法提前返回。

* 对于注入器存在不同的故障处理方法，`require`是最有效的设置，你应该经常使用它。

值得一提的是，回调注入器不能或不应该这样做：

* 回调注入器**不将处理方法代码注入目标**，它们只注入*回调*。如果你想*返回*，请使用*可取消的注入*。

* 回调注入器**不能随意注入构造方法**，只有`RETURN`对构造方法注入器有效。

### 9. 接下来做什么？

回调注入器仅是Mixin提供的最基本的注入器形式。在下列的教程文章中，我们将介绍其他更专业的注入器：

* __[用回调注入器捕获局部变量](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Capture-Locals)__<br />
A secondary feature of regular Callback Injectors, not covered in this introduction, is the ability to capture the local variables at the target location. In this article I introduce the concept of local capture, and the use of surrogates and argument coercion.

* __[介绍重定向注入器](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Redirect-Injectors)__<br />
Probably the most powerful injector type. Redirect injectors allow a target method call or field access to be "redirected" to a custom callback. This type of injector can be leveraged extremely effectively to "wrap around" a method call, change the return value of a method call, or inhibit a method call altogether.

* __[使用ModifyArg注入器修改方法调用参数](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyArg-Injectors)__<br />
**Redirect**'s baby brother, this type of injector allows a single argument to a method to be altered on-the-fly using a callback method.

* __[使用ModifyVariable注入器修改方法中的局部变量](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyVariable-Injectors)__<br />
Whilst delicate and one of the more tricky injectors to use, **ModifyVariable** injectors are the only injector type which can directly edit the value of a local variable within a method.

* __[使用ModifyConstant注入器挂钩和修改文本值](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyConstant-Injectors)__<br />
This type of injector can be used to turn a constant value used in a method into a method call to a callback. Extremely useful for hooking into loop logic, conditionals, or other "hard coded" parts of a target method that you wish to alter.
