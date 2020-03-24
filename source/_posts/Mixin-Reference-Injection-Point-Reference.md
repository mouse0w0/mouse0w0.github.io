---
title: 'Mixin参考——注入点参考'
date: 2020-03-24 14:03:15
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Injection Point Reference](https://github.com/SpongePowered/Mixin/wiki/Injection-Point-Reference)

### 快速导航
<table width="100%">
    <tr><th>简写名</th><th>全名</th></tr>
    <tr><td><a href="#head">HEAD</a></td><td>org.spongepowered.asm.mixin.injection.points.MethodHead</td></tr>
    <tr><td><a href="#return">RETURN</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeReturn</td></tr>
    <tr><td><a href="#tail">TAIL</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeFinalReturn</td></tr>
    <tr><td><a href="#invoke">INVOKE</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeInvoke</td></tr>
    <tr><td><a href="#invokestring">INVOKE_STRING</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeStringInvoke</td></tr>
    <tr><td><a href="#invokeassign">INVOKE_ASSIGN</a></td><td>org.spongepowered.asm.mixin.injection.points.AfterInvoke</td></tr>
    <tr><td><a href="#field">FIELD</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeFieldAccess</td></tr>
    <tr><td><a href="#new">NEW</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeNew</td></tr>
    <tr><td><a href="#constant">CONSTANT</a></td><td>org.spongepowered.asm.mixin.injection.points.BeforeConstant</td></tr>
</table>

<a name="head"></a>
## 在方法起始
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>HEAD</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.MethodHead</tt></td></tr>
> <tr><td><b>选项</b></td><td><i>无</i></td></tr>
> <tr><td><b>参数</b></td><td><i>无</i></td></tr>
> </table>
> 
> ### 描述
> 此注入点始终返回候选列表中的第一条指令。在方法中选择时，始终是方法中的第一条指令；在指令片段（slice）中选择时，始终是指令片段中的第一条指令。
> ### 注释
> 此注入点保证只选择一条指令。

---
<a name="return"></a>
## 在返回之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>RETURN</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeReturn</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl><dt>ordinal</dt><dd>要选中的<tt>RETURN</tt>指令的序号（从<tt>0</tt>开始索引）</dd></dl>
> </td></tr>
> <tr><td><b>参数</b></td><td><i>无</i></td></tr>
> </table>
> 
> ### 描述
> 此注入点选择目标指令列表中的<tt>RETURN</tt>（返回）操作符。如果未指定<tt>ordinal</tt>（序号），则选择所有<tt>RETURN</tt>指令。
>
> <table><thead><tr><th>序号</th><th>检索范围</th><th>结果</th><th>原因</th></tr></thead>
> <tbody>
> <tr><td>省略的</td><td>方法</td><td>1个或更多指令</td><td>方法保证至少有1个<tt>RETURN</tt></tr>
> <tr><td>指定的</td><td>方法</td><td>0或1个指令</td><td>与<tt>ordinal</tt>匹配的<tt>RETURN</tt>可能不存在</td></tr>
> <tr><td>省略的</td><td>指令片段</td><td>0个或更多指令</td><td>匹配指令片段中的所有<tt>RETURN</tt>，其中也可能不包含任何<tt>RETURN</tt></td></tr>
> <tr><td>指定的</td><td>指令片段</td><td>0或1个指令</td><td>匹配指令片段中的一个<tt>RETURN</tt>，其中也可能不包含期望的指令</td></tr>
> </tbody></table>
>
> ### 注释
> 这是<tt>&#64;Inject</tt>注入器注入构造函数时唯一允许使用的注入点。
> 
> 当与[可取消的非空注入](https://github.com/SpongePowered/Mixin/wiki/Advanced-Mixin-Usage---Callback-Injectors#5-targets-with-non-void-return-types)结合使用时，此注入点将使返回类型在<tt>CallbackInfoReturnable</tt>中通过<tt>getReturnType()</tt>方法可用。

---
<a name="tail"></a>
## 在最终返回之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>TAIL</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeFinalReturn</tt></td></tr>
> <tr><td><b>选项</b></td><td><i>无</i></td></tr>
> <tr><td><b>参数</b></td><td><i>无</i></td></tr>
> </table>
> 
> ### 描述
> 此注入点返回目标方法中最后一个`RETURN`操作符。
> ### 注释
> 请注意，最后一个“RETURN”操作符可能与原Java源代码中方法的概念上的“最后”不对应，因为条件表达式可能导致编译得到的字节码与原Java源代码的顺序有显著差异。

---
<a name="invoke"></a>
## 在调用之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>INVOKE</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeInvoke</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>target</dt><dd>被调用的目标，如果被重映射，则必须写上全名</dd>
> <dt>ordinal</dt><dd>要选中的已匹配的调用的索引序号（如果忽略该项，则全选）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>boolean</tt> log</dt><dd>当注入器扫描方法时，在控制台中生成详细的输出，这对诊断不正确的行为或以意外方式注入的注入器非常有帮助</dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 此注入点在目标指令列表中搜索*invoke*指令，并返回与条件匹配的指令。`target`应该是一个可解析的[MemberInfo](http://jenkins.liteloader.com/job/Mixin/javadoc/index.html?org/spongepowered/asm/mixin/injection/struct/MemberInfo.html)，它指定要搜索的调用（省略`target`将返回目标中的所有调用指令。序号指定要选择的调用的索引，如果省略，则返回所有匹配的调用。
> ### 注释
> `ordinal`值是*在与`target`匹配的指令*中选择，因此如果目标方法中有4个调用指令，但只有2个与`target`中指定的值匹配，那么给定`ordinal=1`将选择与`target`匹配的第二个指令，而不是方法中的第二个调用。换句话说，`target`总应用在`ordinal`前。

---
<a name="invokestring"></a>
## 在仅有一字符串参数的Void返回值方法调用之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>INVOKE_STRING</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeStringInvoke</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>target</dt><dd>被调用的目标，如果被重映射，则必须写上全名</dd>
> <dt>ordinal</dt><dd>要选中的已匹配的调用的索引序号（如果忽略该项，则全选）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>string</tt> ldc</dt><dd>要匹配的常量值，请参见描述</dd>
> <dt><tt>boolean</tt> log</dt><dd>当注入器扫描方法时，在控制台中生成详细的输出，这对诊断不正确的行为或以意外方式注入的注入器非常有帮助</dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 如同[BeforeInvoke](#invoke)，此注入点在目标指令列表中搜素*invoke*指令，并返回与条件匹配的指令。此特化版本只匹配接受单个`String`并返回`void`的方法，字符串本身必须是文本字符串，并作为查询过程的一部分进行匹配。
>
> 此查询的主要目的是特定匹配`Profiler::startSection`的调用和只接受文本字符串的类似方法。
>
> 应使用名为`ldc`的参数指定要匹配的字符串常量。
> ### 注释
> 此查询只能用于匹配参数作为字符串文本传递的调用。
>
> 关于`target`与`ordinal`的应用顺序，请参见[BeforeInvoke](#invoke)中的注释。

---
<a name="invokeassign"></a>
## 在调用之后
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>INVOKE_ASSIGN</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.AfterInvoke</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>target</dt><dd>被调用的目标，必须是非void返回类型，如果重映射，则必须写上全名</dd>
> <dt>ordinal</dt><dd>要选中的已匹配的调用的索引序号（如果忽略该项，则全选）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>boolean</tt> log</dt><dd>当注入器扫描方法时，在控制台中生成详细的输出，这对诊断不正确的行为或以意外方式注入的注入器非常有帮助</dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 与`BeforeInvoke`类似，此注入点在目标指令列表中搜索*invoke*指令，并返回符合条件的指令。但是目标指令*必须*返回一个值。如果满足此条件，则注入点立即返回调用之后的指令。如果调用紧接着是的`STORE`指令（例如，调用的结果储存在一个局部变量中），则注入点返回紧跟在`STORE`指令之后的指令。
> ### 注释
> 此注入点以与`BeforeInvoke`完全相同的方式选择候选指令，因此注意事项相同。

---
<a name="field"></a>
## 在字段访问之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>FIELD</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeFieldAccess</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>target</dt><dd>被访问的目标字段，如果重映射，则必须写上全名</dd>
> <dt>opcode</dt><dd>要搜索的指令操作符（必须是<tt>GETSTATIC</tt>、<tt>PUTSTATIC</tt>、<tt>GETFIELD</tt>或<tt>PUTFIELD</tt>中的一个），如未指定则匹配所有字段访问</dd>
> <dt>ordinal</dt><dd>要选择的字段访问的索引序号（如果省略，则全选）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>boolean</tt> log</dt><dd>当注入器扫描方法时，在控制台中生成详细的输出，这对诊断不正确的行为或以意外方式注入的注入器非常有帮助</dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 此注入点在目标指令列表中搜索`GETFIELD`和`PUTFIELD`指令（及其静态的等效指令），并返回符合条件的指令。`target`应该是一个可解析的[MemberInfo](http://jenkins.liteloader.com/job/Mixin/javadoc/index.html?org/spongepowered/asm/mixin/injection/struct/MemberInfo.html)，它指定要搜索的字段名（省略`target`将返回目标中相应类型的所有访问指令）。序号指定要选择的指令的索引，如果省略，则返回所有匹配的字段访问指令。
> ### 注释
> `ordinal`值是*在与`target`匹配的指令*中选择，因此如果目标方法中有4个字段访问指令，但只有2个与`target`中指定的值匹配，那么给定`ordinal=1`将选择与`target`匹配的第二个指令，而不是方法中的第二个字段访问。换句话说，`target`总应用在`ordinal`前。

---
<a name="new"></a>
## 在调用对象构造方法之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>NEW</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeNew</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>target</dt><dd>要重映射的构造方法描述符，改描述符必须<b>仅</b>包含<i>类名</i>或<i>类名和构造方法签名</i></dd>
> <dt>ordinal</dt><dd>要选择的NEW操作符的索引序号（如果省略，则为全部）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>String</tt> class</dt><dd>为后向兼容的替代<tt>target</tt>（ <tt>NEW</tt>最初仅支持此参数）。与<tt>target</tt>相同，仅能指定一个或另一个构造方法</dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 此注入点在目标指令列表中搜索`NEW`指令，并返回符合条件的指令。如果指定了`target`，则只返回指定类型的`NEW`指令。如果`target`包含方法签名（该签名**必须**指定void（`V`)为返回类型），则只返回后跟有给定签名的<tt>INVOKESPECIAL(&lt;init&gt;)</tt>指令的`NEW`指令。注意，**总是返回`NEW`指令，而不是`INVOKESPECIAL`**。
> ### 注释
> 注意，与其他注入点一样，`ordinal`适用于其他注入点参数的相同结论。因此，如果使用`target`标识`NEW`操作符的子集，`ordinal`将基于此子集进行选择，而不是在目标中的整个`NEW`操作符池中进行选择。在词语顺序中，`ordinal`始终*最后*应用。

---
<a name="constant"></a>
## 在常量使用之前
> ### 概览
> <table> 
> <tr><td nowrap><b>简写名</b></td><td><tt>CONSTANT</tt></td></tr>
> <tr><td><b>全名</b></td><td><tt>org.spongepowered.asm.mixin.injection.points.BeforeConstant</tt></td></tr>
> <tr><td><b>选项</b></td><td>
> <dl>
> <dt>ordinal</dt><dd>要选择的NEW操作符的索引序号（如果省略，则为全部）</dd>
> </dl>
> </td></tr>
> <tr><td><b>参数</b></td><td>
> <dl>
> <dt><tt>boolean</tt> nullValue</dt><dd>将此值设置为<tt>true</tt>以匹配方法体中的<tt>null</tt></dd>
> <dt><tt>int</tt> intValue</dt><dd>将此值设置为要在方法体中匹配的整数。请注意，如果要匹配构成条件表达式中的一部分0值，可能还需要设置<tt>expandZeroConditions</tt>参数</dd>
> <dt><tt>float</tt> floatValue</dt><dd>将此值设置为要在方法体中匹配的单精度浮点数</dd>
> <dt><tt>long</tt> longValue</dt><dd>将此值设置为要在方法体中匹配的长整数</dd>
> <dt><tt>double</tt> doubleValue</dt><dd>将此值设置为要在方法体中匹配的双精度浮点数</dd>
> <dt><tt>String</tt> stringValue</dt><dd>将此值设置为要在方法体中匹配的文本</dd>
> <dt><tt>String</tt> class</dt><dd>将此值设置为要在方法体中匹配的<tt>Class</tt></dd>
> <dt><tt>boolean</tt> log</dt><dd>将此值设置为<tt>true</tt>以在应用此注入点查询时发出日志信息</dd>
> <dt><tt>String</tt> expandZeroConditions</dt><dd>虽然大多数常量可以相对容易地位于编译后的方法中，但在条件表达式中使用0时存在特殊情况。例如：<blockquote><code>if (x >= 0)</code></blockquote>出现这种特殊情况是因为Java包含用于这种类型的比较的显式指令，因此编译后的代码可能更像这样：<blockquote><code>if (x.isGreaterThanOrEqualToZero())</code></blockquote>当然如果我们知道正在搜索的常量是比较表达式的一部分，那么我们可以显式地搜索<tt>isGreaterThanOrEqualToZero</tt>并将其转换为原始形式，以便像任何其他常量访问一样重定向它<br /><br />如要启用此行为，可以基于要展开的表达式类型为次参数指定一个或多个值。因为Java编译器习惯将某些表达式编译为与其源代码对应的<i>逆</i>表达式（例如，编译<i>当大于某值时做某事</i>结构为<i>如果小于或等于某值则不做某事</i>）；指定特定的表达式类型也隐式包括逆表达式。<br /><br />值得注意的是，其对序号的影响可能很难预测，因此应注意确保所选注入点与预期位置匹配<br /><br />指定此选项具有以下效果：<br /><br /><ul><li>目标方法中的条件匹配操作符被识别为注入候选</li><li><tt>intValue</tt>为<tt>0</tt>是隐式的，不需要显式定义</li><li>但是，显式指定<tt>intValue</tt>为<tt>0</tt>将导致此选择器也与方法体重的显示<tt>0</tt>常量匹配</li></ul>要为该选项指定值，请用逗号或空格分隔<tt>条件值</tt>，例如：<blockquote><code>"expandZeroConditions=LESS_THAN_ZERO,GREATER_THAN_ZERO"</code></blockquote></dd>
> </dl>
> </td></tr>
> </table>
> 
> ### 描述
> 此注入点在目标指令列表中搜索字面值（常量），并返回与条件相匹配的指令。请注意，所有鉴别参数都是互斥的，并且只应指定*一个*鉴别参数。指定多个鉴别参数将抛出异常。
> ### 注释
> 注意，与其他注入点一致，`ordinal`适用于注入点其他参数的总体结果。在词语顺序中，`ordinal`始终*最后*应用。
