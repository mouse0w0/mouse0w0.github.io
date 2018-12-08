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

So far, we have only considered injecting into a target method which returns `void`. When injecting into a method with a non-`void` return type, injector **handler methods** will instead need to receive a `CallbackInfoReturnable`. The `CallbackInfoReturnable` differs from its parent `CallbackInfo` in that:

* `CallbackInfoReturnable` is generic, with the return type of the method as its type argument
* When *cancelling* a `cancellable` callback with a return type, the value to return from the method can be specified by calling `setReturnValue` instead of `cancel`

As you can imagine, this is incredibly useful when injecting into getters or when **Short Circuiting** a method with an inject at `HEAD`. Let's take a look at an example method which returns a value:

![](https://raw.githubusercontent.com/SpongePowered/Mixin/master/docs/images/flard_10.png)

The `getPos` method is a typical *getter* method in that its method body simply returns the value of a protected field. Let's add a mixin which injects at `HEAD` and returns a defaulted value if the field is `null`:

![](https://raw.githubusercontent.com/SpongePowered/Mixin/master/docs/images/flard_11.png)

The injector first merges our **handler method** into the target class, next it injects code into the **target method** to handle the cancellation.

![](https://raw.githubusercontent.com/SpongePowered/Mixin/master/docs/images/flard_12.png)

Note how the injected code differs from the `void`-type short-circuit code shown above. This time the injector returns the value we set in the `CallbackInfoReturnable` if the callback is cancelled. Our mixin code looks like this

```java
@Inject(method = "getPos", at = @At("HEAD"), cancellable = true)
protected void onGetPos(CallbackInfoReturnable<Point> cir) {
    if (this.position == null) {
        // setReturnValue implicitly cancel()s the callback
        cir.setReturnValue(Point.ORIGIN);
    }
    
    // Note that if the handler returns normally then the method
    // continues as normal, just like a normal cancellable
    //  injection does if cancel() is not called.
}
```

As you can imagine, this type of injector is extremely powerful, and is not restricted to `HEAD` injections, you can use any Injection Point that you wish. Something special happens when using cancellable returnable injections on `RETURN` however:

You may be thinking *"but surely in this very simple method, the `HEAD` and `RETURN` basically mean the same thing, right?"*

You would be forgiven for thinking that, since it seems like the logical deduction, however in practice the single statement `return this.position` actually consists of two operations:

1. Fetch the value of the field `position`
2. Return the value

The value is temporarily stored in the method's *argument stack*, you can think of these as temporary, invisible variables which store values as the JVM manipulates them. What this means from a practical perspective is that `HEAD` and `RETURN` are actually separate places in the method!

![](https://raw.githubusercontent.com/SpongePowered/Mixin/master/docs/images/flard_13.png)

Since we know that a `RETURN` opcode must return a value, we know that the value is available for us to access whenever we inject at `RETURN`. The injection processor handles this situation and "captures" the return value into the CallbackInfo passed to our handler:

![](https://raw.githubusercontent.com/SpongePowered/Mixin/master/docs/images/flard_14.png)

This approach has a number of benefits:

* The contract of our injector can now be more loosely coupled to the implementation of the **target** method. In other words, if the **target** method changes, our handler doesn't need to know about the change, since it only cares about the value being returned.
* If a **target** method has multiple `RETURN` opcodes, the value returned can still be processed without needing additional context
* It allows us to make "observer" injections which only *inspect* the value being returned without actually altering it or needing to worry about the method's implementation.

For example, let's say we alter our example injector above to use `RETURN` instead of `HEAD`. All we care about is having the method not return `null`, so our code becomes:

```java
@Inject(method = "getPos", at = @At("RETURN"), cancellable = true)
protected void onGetPos(CallbackInfoReturnable<Point> cir) {
    // Check the captured return value
    if (cir.getReturnValue() == null) {
        // if it's null, set our fallback value as the return
        cir.setReturnValue(Point.ORIGIN);
    }
}
```

### 6. Callback Injector features

#### 6.1 Multiple Injection Points

It should be noted that the `at` parameter of an `@Inject` annotation is actually an array type. This means that it is quite possible to specify multiple Injection Points for a single injector **handler** method.

#### 6.2 Wildcard Targets

When specifying the target `method` for an injector, appending an asterisk (`*`) the method name directs the mixin processor to match all methods with the specified name, regardless of their signature. This allows a single injector to target multiple methods.

Using this syntax makes it impossible to capture method arguments, since the method arguments will vary from method to method. However it makes observer-type injections trivial. For example, consider the following injection:

```java
@Inject(method = "<init>*", at = @At("RETURN"))
private void onConstructed(CallbackInfo ci) {
    // do initialisation stuff
}
```

This injector injects a callback into all constructors in the class. This is useful if a target class has multiple overloaded constructors and you simply wish to inject into all of them.

Wildcard targets can be used with any method name, however if a wildcard matches methods with both `void` return types and non-`void` return types, then the injection will fail with an error because a `CallbackInfoReturnable` is required for the non-void targets.

### 7. Considerations and Limitations for Callback Injectors

#### 7.1 Injecting into constructors

Now constructors in java code are fairly straightforward and impose three simple restrictions:

1. you must call `super` before any other code
2. you must initialise any `final` fields.
3. as a corollary to point 1, you cannot invoke an instance method inline in the `super` call, any calls made here must be static

However at the *bytecode* level constructors are much more delicate. Since a compiled `<init>` method represents a mish-mash of the original constructor code, any field initialisers for the class (duplicated in all constructors), and in some cases synthetic (compiler-generated) code as well (for example in `Enum` constructors). Because of their nature, they represent a minefield for bytecode-level transformations.

Mixin thus imposes the following restriction on injecting into constructors:

* **The only Injection Point supported for constructors is the `RETURN` injector**
 This restriction is imposed because there is no other sensible way to be sure that the class is fully initialised before calling your handler code.

If you want to inject into a constructor, you **must** specify `RETURN` as your injection point.

#### 7.2 Failure States

Like other Mixin capabilities, Callback Injectors are designed to be *fail-fast* and *fail-safe*. This generally means that if an injector fails it will generally do one of two things:

1. Fail "silently" (unless mixin debugging is enabled) and leave the **target** method untouched (however the **handler** method will still be merged into the target class)
2. Fail with a deterministic error message: for example when an injector finds its target opcode but the **handler** method has an incorrect signature.

Injectors are designed so that any failure will **not** corrupt the method bytecode, it will either succeed or deterministically fail.

Silent failure is usually reserved for Injectors whose Injection Points match no opcodes. This can happen when the target method is altered or replaced and can be extremely useful since "overloaded" injectors can be created to cater for different situations.

However sometimes it may be important that a certain injector succeeds. Perhaps the stability of your application depends on it, or failure is not an anticipated state and the application should be shut down. Either way, it is sometimes necessary to be able to *insist* that an injector succeeds (and raise a failure state if it does not). This is possible using the `expect` and `require` arguments.

##### 7.2.1 Require

The `require` argument is simple, specifying a value for `require` declares that the injector must succeed *at least this many times*.

```java
@Inject(method = "foo", at = @At(value = "INVOKE", target = "someMethod"), require = 2)
private void onInvokeSomeMethodInFoo(CallbackInfo ci) {
    ...
```

In this example, we expect our Injection Point to match 2 invocations in the **target** method. Specifying `require` means that if fewer than 2 injections result, then an error will be raised.

It is also possible to specify a config-wide value for `require` in your mixin config file.

##### 7.2.2 Expect

Sometimes failing at runtime is okay, but when developing your mixins you want to be able to use the `require` functionality (error when an injection fails) without having to sprinkle `require` everywhere and then remember to remove it before your code goes into production!

The `expect` argument allows you to do exactly that. `expect` works exactly like `require`, apart from the fact that it is only processed whenever the `mixin.debug.countInjections` system property is set to `true`. This allows you to check your injectors are working at dev time, but allow them to fail-fast in production.

##### 7.2.3 Global Injector Settings

You can specify a configuration-wide default value for `require` in your mixin config file, by including an `injectors` block, you can override the default value for `require` for all mixins in your config:

```json
{
   "injectors": {
        "defaultRequire": 1
    }
}
```

##### 7.2.4 Injector Groups

Whilst the semantics of `require` allow you to stipulate behaviour for a single injector, you may encounter a situation where you wish to provide multiple, alternative injectors for a single case. This is useful if your Mixin targets multiple environments, or if another transformer is known to alter a particular target method in a well-defined way.

In these circumstances you may provide two or more injectors in which only one is expected to succeed for a given environment. This of course presents a problem with how you might leverage the `require` value for those injectors, since if at least one is *expected* to fail in a given scenario this makes the use of `require` impossible.

To tackle this situation, declaration of injector *groups* is supported. Using injector groups allows a `min` and `max` number of injections to be specified *for the group*, thus ensuring that the injections can still be verified as working but only the specified number of injections should occur.

To leverage injector groups, simply decorate each injector **handler method** with an `@Group` annotation. The first annotation should also specify `min` and (optionally) `max`. If multiple injector groups are used in a single mixin, then a unique `name` for the group should be specified on all of the `@Group` annotations in the mixin.

#### 7.3 Override Behaviour for Handler Methods

When you define a Callback Injector **handler** method, your method is merged into the **target class** using normal mixin behaviour. However this has implications for sub-classing, in particular injectors defined in derived types.

* **Handler methods are renamed before merging** - all **handler** methods are *decorated* before being merged into the target class. This ensures that if another mixin to the *same* class defines an *identical* injector, the two handler methods will not conflict.

* **Handler methods are called using the opcode matching their access level** - if your **handler** method is `private` then it will be called using `INVOKESPECIAL` (static binding), if your **handler** method is non-private it will be called using `INVOKEVIRTUAL`, this will allow you to `@Override` the handler in a derived mixin.

* **If you `@Override` a handler method in a derived mixin, it will be renamed to match the decoration of its supermixin counterpart** - this is done so that a method in a derived class can never "accidentally" override your **handler** method.

In general, unless explicitly planning to use override semantics for a particular handler, it is recommended that **handler** methods be `private`.

### 8. Summary

This might seem like a lot to take in, Callback Injectors are powerful and quite nuanced, and as such that's not an unreasonable way to feel! Let's recap the key points to put this all in perspective:

* Callback Injectors are just regular mixin methods which have the special behaviour of *injecting* a callback to themselves somewhere else in the target class

* They always consist of a **handler method**, a **target method** and some *Injection Points* inside the target

* Callback Injector methods always take a `CallbackInfo`, and can take other arguments, such as the arguments of the **target** method, as well

* Callbacks can be *cancellable*, allowing a premature return from the target method

* Different ways of handling failure exist for injectors, `require` is the most useful setting and you should use it often

It's also worth mentioning things that callback injectors cannot, or do not, do:

* Callback Injectors **do not inject the handler code into the target**, they only ever inject a *callback*. If you want to *return*, use a *cancellable injection*.

* Callback Injectors **cannot inject arbitrarily into constructors**, only `RETURN` is valid for constructor injectors

### 9. What Comes Next?

Callback Injectors are the most basic form of injector provided by Mixin. In the following tutorial articles, we will introduce the other, more specialised, injectors:

* __[Capturing local variables with Callback Injectors](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Capture-Locals)__<br />
A secondary feature of regular Callback Injectors, not covered in this introduction, is the ability to capture the local variables at the target location. In this article I introduce the concept of local capture, and the use of surrogates and argument coercion.

* __[Introduction to Redirect Injectors](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Redirect-Injectors)__<br />
Probably the most powerful injector type. Redirect injectors allow a target method call or field access to be "redirected" to a custom callback. This type of injector can be leveraged extremely effectively to "wrap around" a method call, change the return value of a method call, or inhibit a method call altogether.

* __[Using ModifyArg Injectors to modify method invocation arguments](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyArg-Injectors)__<br />
**Redirect**'s baby brother, this type of injector allows a single argument to a method to be altered on-the-fly using a callback method.

* __[Tweaking local variables in a method using ModifyVariable Injectors](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyVariable-Injectors)__<br />
Whilst delicate and one of the more tricky injectors to use, **ModifyVariable** injectors are the only injector type which can directly edit the value of a local variable within a method.

* __[Hooking and modifying literal values using ModifyConstant Injectors](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---ModifyConstant-Injectors)__<br />
This type of injector can be used to turn a constant value used in a method into a method call to a callback. Extremely useful for hooking into loop logic, conditionals, or other "hard coded" parts of a target method that you wish to alter.
