---
title: 介绍 Nashorn —— Java 8 JavaScript 引擎
date: 2018-12-02 19:07:03
tags: [Java]
---
本文我们来介绍一下Java 8的Nashorn JavaScript引擎。Nashorn是于Java 8中用于取代Rhino（Java 6，Java 7）的JavaScript引擎。Nashorn完全支持ECMAScript 5.1规范以及一些扩展。与先前的Rhino引擎相比，它有二到十倍的性能提升。本文中将使用各种各样的例子来说明Nashorn的强大功能。

## jjs
jjs是个基于Nashorn引擎的命令行工具。你可以通过该工具快速地在Java上运行JavaScript代码，就像是一个REPL。

例如，运行一个`hello.js`文件：
```shell
$ $JAVA_HOME/bin/jjs hello.js
Hello World
```

或者，你还可以直接运行代码：
```shell
$ $JAVA_HOME/bin/jjs
jjs> print("Hello World")
Hello World
```

## 在Java中调用Nashorn引擎
本文专注于在Java中调用Nashorn，所以现在在Java代码中实现简单的HelloWorld：
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval("print('Hello World!');");
```

或者，我们还可以从文件中运行JS：
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("hello.js"));
```

## 编译JavaScript代码
你同样可以将脚本编译为Java字节码后调用，这样在多次调用的情况下效率会更高，例如：
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
CompiledScript compiledScript = ((Compilable) engine).complie("print('Hello World!');");
engine.eval();
```

## 传递数据到脚本
数据可以通过定义`Bindings`传递到引擎中：
```java
Bindings bindings = engine.createBindings();
bindings.put("name", "Nashorn");

engine.eval("print('Hello ' + name);");
```

运行该程序将输出`Hello Nashorn`。

## 在Java中调用JavaScript函数
Nashorn支持从Java代码中直接调用定义在脚本中的JavaScript函数。你可以将Java对象作为函数参数传递，并且使用函数返回值调用Java方法。

例如在脚本中定义如下代码：
```js
var fun1 = function(name) {
    print('Hello ' + name);
    return "Hi!";
}

var fun2 = function (object) {
    print(Object.prototype.toString.call(object));
};
```

为了调用函数，你首先需要将脚本引擎转换为`Invocable`接口。`NashornScriptEngine`已经实现了`Invocable`接口，并且定义了`invokeFunction`方法来调用指定名称的JavaScript函数。

```js
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("hello.js"));

Invocable invocable = (Invocable) engine;

Object result = invocable.invokeFunction("fun1", "Nashorn");
System.out.println(result);
System.out.println(result.getClass());
```

最终将输出如下内容：
```
Hello Nashorn
Hi!
java.lang.String
```

Java对象在传入时不会在JavaScript上损失任何类型信息。由于脚本在JVM上运行，我们可以在Nashron上使用Java API或外部库的全部类或方法。

现在让我们传入任意Java对象来调用第二个方法：
```java
invocable.invokeFunction("fun2", new Date());
// [object java.util.Date]

invocable.invokeFunction("fun2", LocalDateTime.now());
// [object java.time.LocalDateTime]

invocable.invokeFunction("fun2", new Person());
// [object my.package.Person]
```

## 调用Java静态方法和字段
在JavaScript中调用Java方法非常容易，就像在Java中所做的一样。首先我们先定义一个Java静态方法：
```java
public static String sayHello(String name) {
    System.out.println("Hello " + name);
    return "Hi!";
}
```

然后Java类就可以通过`Java.type`API在JavaScript中引用，就像Java的`import`一样，例如：
```js
var MyJavaClass = Java.type(`my.package.MyJavaClass`);

var result = MyJavaClass.sayHello('Nashorn');
print(result);
```

最终的结果是：
```
Hello Nashorn
Hi!
```

为了理解在使用JavaScript原生类型调用Java方法时，Nashorn是如何处理类型转换的。我们将通过简单的例子来展示：

下面的方法将打印实际的参数类型：
```java
public static void fun(Object obj) {
    System.out.println(obj.getClass());
}
```

接下来我们用JavaScript来调用该方法：
```js
MyJavaClass.fun(127);
// class java.lang.Integer

MyJavaClass.fun(49.99);
// class java.lang.Double

MyJavaClass.fun(true);
// class java.lang.Boolean

MyJavaClass.fun("Hi!");
// class java.lang.String

MyJavaClass.fun(new Number(127));
// class jdk.nashorn.internal.objects.NativeNumber

MyJavaClass.fun(new Date());
// class jdk.nashorn.internal.objects.NativeDate

MyJavaClass.fun(new RegExp());
// class jdk.nashorn.internal.objects.NativeRegExp

MyJavaClass.fun({foo: 'bar'});
// class jdk.nashorn.internal.scripts.JO4
```

## 创建Java对象
创建Java对象也如图调用Java方法一样简单，代码如下：
```js
var HashMap = Java.type(`java.util.HashMap`);
var mapDef = new HashMap();
var map100 = new HashMap(100);
```

## 访问Java类的补充说明
同样，访问Java类不一定需要`Java.type`函数，可直接书写类名访问。例如：
```js
var result = my.package.MyJavaClass.sayHello('Nashorn');
print(result);

var mapDef = new java.util.HashMap();
```

同样，为了方便，Nashorn默认提供了对几个Java包的访问，分别是：`com`、`edu`、`java`、`javafx`、`javax`和`org`。
```
jjs> java.lang
[JavaPackage java.lang]
jjs> typeof java.lang
object
jjs> java.lang.System
[JavaClass java.lang.System]
jjs> typeof java.lang.System
function
jjs> typeof java.lang.System == "function"
true
```

## 语言扩展
Nashorn虽然是面向ECMAScript 5.1实现的但它提供了一些扩展，使JavaScript能更好的运用。

### 类型数组
JavaScript的原生数组是无类型的。Nashron允许你在JavaScript中使用Java的类型数组：
```js
var IntArray = Java.type("int[]");

var array = new IntArray(5);
array[0] = 5;
array[1] = 4;
array[2] = 3;
array[3] = 2;
array[4] = 1;

try {
    array[5] = 23;
} catch (e) {
    print(e.message);  // Array index out of range: 5
}

array[0] = "17";
print(array[0]);  // 17

array[0] = "wrong type";
print(array[0]);  // 0

array[0] = "17.3";
print(array[0]);  // 17
```

`int[]`数组就像真实的Java整数数组那样。但在试图向数组添加非整数时，Nashron执行了一些隐式的转换。字符串会自动转换为整数，这十分便利。

### 用foreach语句迭代数组或集合
我们可以在JavaScript使用foreach语句迭代数组或集合：
```js
var list = [1, 2, 3, 4, 5];
  var result = '';
  for each (var i in list) {
  result+=i+'-';
  }
  print(result);

var ArrayList = Java.type('java.util.ArrayList');
var list = new ArrayList();
list.add('a');
list.add('b');
list.add('c');

for each (var el in list) {
    print(el);  // a, b, c
}

var map = new java.util.HashMap();
map.put('foo', 'val1');
map.put('bar', 'val2');

for each (var e in map.keySet()) {
    print(e);  // foo, bar
}

for each (var e in map.values()) {
    print(e);  // val1, val2
}
```

### 函数字面量
在简单的函数声明中，可以省略括号
```js
function increment(in) ++in
```

### 条件捕获语句
可以添加特定的catch字句，这些字句仅在指定条件为真时才执行：
```js
try {
    throw "BOOM";
} catch(e if typeof e === 'string') {
    print("String thrown: " + e);
} catch(e) {
    print("this shouldn't happen!");
}
```

### 用Object.setPrototypeOf设置对象原型
Nashorn定义了一个API扩展，它使我们能够更改对象的原型：
```js
Object.setPrototypeOf(obj, newProto);
```

一般认为该函数是对`Object.prototype.__proto__`的一个更好选择，因为它应该是在所有代码中设置对象原型的首选方法。

### Lambda表达式和数据流
每个人都热爱lambda和数据流 -- Nashron也一样！虽然ECMAScript 5.1没有Java8 lmabda表达式的简化箭头语法，我们可以在任何接受lambda表达式的地方使用函数字面值。
```js
var list = new java.util.ArrayList();
list.add("a1");
list.add("a2");
list.add("a3");
list.add("b1");
list.add("b2");
list.add("b3");
list.add("c1");
list.add("c2");
list.add("c3");


list
    .stream()
    .filter(function(el) {
        return el.startsWith("a");
    })
    .sorted()
    .forEach(function(el) {
        print(el);
    });
    // a1, a2, a3
```

### 类的继承
Java类型可以由`Java.extend`轻易继承。如下所示，你甚至可以在脚本中创建多线程的代码：
```js
var Runnable = Java.type('java.lang.Runnable');
var Printer = Java.extend(Runnable, {
    run: function() {
        print('printed from a separate thread');
    }
});

var Thread = Java.type('java.lang.Thread');
new Thread(new Printer()).start();

new Thread(function() {
    print('printed from another thread');
}).start();

// printed from a separate thread
// printed from another thread
```

### 函数重载
方法和函数可以通过点运算符或方括号运算符来调用：
```js
var System = Java.type('java.lang.System');
System.out.println(10);              // 10
System.out["println"](11.0);         // 11.0
System.out["println(double)"](12);   // 12.0
```

当使用重载参数调用方法时，传递可选参数类型`println(double)`会指定所调用的具体方法。

### Java Beans
你可以简单地使用属性名称来向Java Beans获取或设置值，不需要显式调用读写器：
```js
var Date = Java.type('java.util.Date');
var date = new Date();
date.year += 1900;
print(date.year);  // 3918
```

### 属性绑定
两个不同对象的属性可以绑定到一起：
```js
var o1 = {};
var o2 = { foo: 'bar'};

Object.bindProperties(o1, o2);

print(o1.foo);    // bar
o1.foo = 'rab';
print(o2.foo);    // rab
```

### 字符串扩展
Nashorn在String原型上提供了两个简单但非常有用的扩展。这就是`trimRight`和`trimLeft`函数，它们可返回String得副本并删除空格：
```js
print("   hello world".trimLeft());
print("hello world     ".trimRight());
```

### 位置
当前文件名，目录和行可以通过全局变量`__FILE__`、`__LINE__`和`__DIR__`获取：
```js
print(__FILE__, __LINE__, __DIR__);
```

### 导入作用域
有时一次导入多个Java包会很方便。我们可以使用`JavaImporter`类，和`with`语句一起使用。所有被导入包的类文件都可以在`with`语句的局部域中访问到。
```js
var imports = new JavaImporter(java.io, java.lang);
with (imports) {
    var file = new File(__FILE__);
    System.out.println(file.getAbsolutePath());
}
```

### 数组转换
下面的代码将Java的`List`转换为JavaScript原生数组：
```js
var javaList = new java.util.ArrayList();
javaList.add("1");
javaList.add("2");
javaList.add("3");
var jsArray = Java.from(javaList);
print(jsArray);                                  // 1, 2, 3
print(Object.prototype.toString.call(jsArray));  // [object Array]
```

下面的代码执行相反操作：
```js
var javaArray = Java.to([3, 5, 7, 11], "int[]");
```

### 访问超类
在JavaScript中访问被覆盖的成员通常比较困难，因为Java的super关键字在ECMAScript中并不存在。幸运的是，Nashron有一套补救措施。

首先我们需要在Java代码中定义超类：

```java
class SuperRunner implements Runnable {
    @Override
    public void run() {
        System.out.println("super run");
    }
}
```

下面我在JavaScript中覆盖了SuperRunner。要注意创建新的Runner实例时的Nashron语法：覆盖成员的语法取自Java的匿名对象。
```js
var SuperRunner = Java.type('my.package.SuperRunner');
var Runner = Java.extend(SuperRunner);

var runner = new Runner() {
    run: function() {
        Java.super(runner).run();
        print('my run');
    }
}
runner.run();

// super run
// my run
```

我们通过Java.super()扩展调用了被覆盖的SuperRunner.run()方法。

### 神奇的__noSuchProperty__和__noSuchMethod__
可以在对象上定义方法，每当访问未定义属性或调用未定义方法时，将调用该方法：
```js
var demo = {
    __noSuchProperty__: function (propName) {
        print("Accessed non-existing property: " + propName);
    },
     
    __noSuchMethod__: function (methodName) {
        print("Invoked non-existing method: " + methodName);
    }
};
 
demo.doesNotExist;
demo.callNonExistingMethod()
```

这将输出：
```
Accessed non-existing property: doesNotExist
Invoked non-existing method: callNonExistingMethod
```

### Java.asJSONCompatible 函数
使用该函数，我们可以得到一个与Java JSON库期望兼容的对象。代码如下：
```java
Object obj = engine.eval("Java.asJSONCompatible(
  { number: 42, greet: 'hello', primes: [2,3,5,7,11,13] })");
Map<String, Object> map = (Map<String, Object>)obj;
  
System.out.println(map.get("greet"));
System.out.println(map.get("primes"));
System.out.println(List.class.isAssignableFrom(map.get("primes").getClass()));
```

这将输出：
```
hello
[2, 3, 5, 7, 11, 13]
```

### 载入脚本
你可以在脚本引擎中载入其他JavaScript文件：
```js
load('classpath:script.js');
```

或者通过URL载入脚本：
```js
load('/script.js');
```

请记住，JavaScript没有命名空间的概念，所以所有的内容都堆放在全局环境中。这使得加载的脚本有可能与你的代码或它们之间的命名冲突。这可以使用`loadWithNewGlobal`函数尽可能减少这种情况的发生：
```js
var math = loadWithNewGlobal('classpath:math_module.js')
math.increment(5);
```

`math_module.js`的文件内容如下：
```js
var math = {
    increment: function(num) {
        return ++num;
    }
};
 
math;
```

这里我们定义了一个名为`math`的对象，其中有一个名为`increment`的函数。使用这个范例我们可以达成基本的模块化。

## ScriptObjectMirror
在向Java传递原生JavaScript对象时，你可以使用`ScriptObjectMirror`类，它实际上是底层JavaScript对象的Java表示。`ScriptObjectMirror`实现了`Map`接口，其位于`jdk.nashorn.api`中。这个包中的类可以用于Java代码。

下面的例子将参数类型从`Object`改为`ScriptObjectMirror`，我们可以从传入的JavaScript对象中获得一些信息。
```java
static void fun(ScriptObjectMirror mirror) {
    System.out.println(mirror.getClassName() + ": " +
        Arrays.toString(mirror.getOwnKeys(true)));
}
```

当向这个方法传递对象（哈希表）时，在Java中可以访问其属性：
```js
MyJavaClass.fun({
    foo: 'bar',
    bar: 'foo'
});

// Object: [foo, bar]
```

我们也可以在Java中调用JavaScript的函数。让我们首先在JavaScript中定义一个`Person`类型，其含有属性`firstName`和`lastName`，以及函数`getFullName`。
```js
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.getFullName = function() {
        return this.firstName + " " + this.lastName;
    }
}
```

JavaScript方法`getFullName`可以通过`callMember()`在`ScriptObjectMirror`上调用。
```java
static void fun(ScriptObjectMirror person) {
    System.out.println("Full Name is: " + person.callMember("getFullName"));
}
```

当向Java方法传递新的`Person`时，我们会在控制台看到预计的输出：
```js
var person = new Person("Peter", "Parker");
MyJavaClass.fun(person);

// Full Name is: Peter Parker
```

## 限制脚本对特定Java类的访问
`jdk.nashorn.api.scripting.ClassFilter`接口限制通过Nashorn运行的脚本对特定Java类的访问，为JavaScript代码对Java类的访问提供了细粒度的控制。示例代码如下：
```java
import javax.script.ScriptEngine;
import jdk.nashorn.api.scripting.ClassFilter;
import jdk.nashorn.api.scripting.NashornScriptEngineFactory;
 
public class MyClassFilterTest {
 
    class MyCF implements ClassFilter { // 创建类过滤器
        @Override
        public boolean exposeToScripts(String s) {
            if (s.equals("java.io.File")) {
                return false;
            }
            return true;
        }
    }
 
    public void testClassFilter() {
    
        final String script =
        "print(java.lang.System.getProperty(\"java.home\"));" +
        "print(\"Create file variable\");" +
        "var File = Java.type(\"java.io.File\");";
    
        NashornScriptEngineFactory factory = new NashornScriptEngineFactory();
    
        ScriptEngine engine = factory.getScriptEngine(new MyClassFilterTest.MyCF());

        try {
            engine.eval(script);
        } catch (Exception e) {
            System.out.println("Exception caught: " + e.toString());
        }
    }
    
    public static void main(String[] args) {
        MyClassFilterTest myApp = new MyClassFilterTest();
        myApp.testClassFilter();
    }
```
最终这会抛出`java.lang.ClassNotFoundException`异常。

## 命令行脚本
如果你对编写命令行（shell）脚本感兴趣，来试一试[Nake](https://github.com/winterbe/nake)吧。Nake是一个Java 8 Nashron的简化构建工具。你只需要在项目特定的`Nakefile`中定义任务，之后通过在命令行键入`nake -- myTask`来执行这些任务。任务编写为JavaScript，并且在Nashron的脚本模式下运行，所以你可以使用你的终端、JDK8 API和任意Java库的全部功能。

对Java开发者来说，编写命令行脚本是前所未有的简单...

> 本文部分内容来自：
>
> https://wizardforcel.gitbooks.io/modern-java/content/ch3.html
> 
> https://docs.oracle.com/javase/10/nashorn/nashorn-java-api.htm
>
> https://www.baeldung.com/java-nashorn