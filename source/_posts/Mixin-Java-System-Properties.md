---
title: 'Mixin参考——Mixin Java系统属性'
date: 2022-03-01 15:00:00
tags: [Java, Bytecode, Mixin]
categories: Mixin
---
> 本文翻译自：[Mixin Java System Properties](https://github.com/SpongePowered/Mixin/wiki/Mixin-Java-System-Properties)

下表汇总了Mixin支持的`Java系统属性`，以启用各种调试和审计功能。将任何属性设置为`true`以启用该选项：

<table width="100%">
  <thead>
    <tr>
      <th>系统属性</th><th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td valign="top"><code>mixin.debug</code></td>
      <td valign="top">启用<b>所有</b>Mixin调试选项</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.export</code></td>
      <td valign="top">
        <code>export</code>调试选项使Mixin处理器为所有Mixin目标向磁盘导出混入后的字节码。字节码被输出到你的运行目录下的<code>.mixin.out</code>文件夹下的典型包/类结构中。
        <p>将Fernflower的jar放在运行时类路径上时也会导致这些类文件被反编译。</p>
      </td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.export.filter</code></td>
      <td valign="top">导出过滤器，如果为缺省值，允许导出所有转换的类。如果指定值，则作为要导出的类名的过滤器，并且只导出匹配的类。这在使用Fernflower时非常有用，否则导出可能非常慢。允许使用以下通配符。
      <dl>
        <dt>*</dt><dd>匹配除点（.）以外的一个或多个字符</dd>
        <dt>**</dt><dd>匹配任意数量的字符</dd>
        <dt>?</dt><dd>仅匹配一个字符</dd>
      </dl></td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.export.decompile</code></td>
      <td valign="top">如果设置为false，即使在类路径上找到Fernflower，也会禁用它。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.export.decompile.async</code></td>
      <td valign="top">在单独的线程中运行Fernflower。一般来说，这将使导出对启动耗时的影响降低（反编译通常会增加大约20%的加载时间），同时注意崩溃可能导致导出未反编译。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.verify</code></td>
      <td valign="top"><code>verify</code>选项在混入后的字节码上运行ASM的<code>CheckClassAdapter</code>以检查是否正确应用了Mixin转换。此选项仅用于Mixin库，不建议在Mixin的常规调试期间启用。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.verbose</code></td>
      <td valign="top"><code>verbose</code>选项将Mixin处理器生成的所有<code>DEBUG</code>级别日志消息提升到<code>INFO</code>级别，以便在运行时将它们发送到控制台。当使用Mixin开发时，这是一个非常有用的选项，因为它允许对Mixin应用进程进行更交互式的监控。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.countInjections</code></td>
      <td valign="top">将失败的注入提升到错误状态。详细信息请参见<a href="http://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/Inject.html#expect--"><code>Inject.expect</code></a>。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.strict</code></td>
      <td valign="top">启用严格检查。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.strict.unique</code></td>
      <td valign="top">如果为false（默认值），<a href="http://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/Unique.html">Unique</a>公共方法仅在遇到问题时发出警告，不会合并到目标中。如果为true，则会抛出异常。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.strict.targets</code></td>
      <td valign="top">启用对Mixin目标的严格检查。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.debug.profiler</code></td>
      <td valign="top">为所有Mixin操作启用性能分析器（通常仅在Mixin准备操作期间启用）。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.dumpTargetOnFailure</code></td>
      <td valign="top">有时，Mixin会失败，并显示一条神秘的消息，例如目标中不存在影子目标，或其他意外错误，这表明目标类未处于特定Mixin预期的状态。有时，这可能是因为另一个转换器正在以Mixin转换器无法预料的方式对字节码进行转换，或者目标类发生了其它意外的更改。启用此选项会导致<code>InvalidMixinException</code>和其他运行时Mixin错误，从而将<i>传入的</i>（未经混入的）类字节码转储到磁盘。这可以使用<code>javap</code>或其它Java反汇编程序检查目标类字节码，以确定不匹配的原因。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.checks</code></td>
      <td valign="top">启用<b>所有</b>Mixin检查操作</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.checks.interfaces</code></td>
      <td valign="top">
        <p>启用<i>接口实现审计模式</i>。启用此模式后，Mixin处理器将为每个应用的Mixin输出一个审计报告，该报告提供了由类方法声明但<i>未由类或任何超类实现</i>的接口方法的摘要，实际上，那些方法在被调用时会导致抛出<code>AbstractMethodError</code>。</p>
        <p>报告被生成到<code>标准错误（STDERR）</code>输出，也会写入你的运行目录下的<code>.mixin.out</code>文件夹下的平面文件。</p>
      </td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.checks.interfaces.strict</code></td>
      <td valign="top">如果启用了接口检查，“严格模式”（默认）会将实现检查应用于抽象目标类。将此选项设置为<code>false</code>会导致在生成实现报告时跳过抽象目标。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.ignoreConstraints</code></td>
      <td valign="top">禁用<em>约束检查</em>，将违反约束的情况从致命错误降级为仅输出警告。用于开发或out-of-band目标的in-the-wild测试</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.hotSwap</code></td>
      <td valign="top">启用热插拔代理。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env</code></td>
      <td valign="top">环境设置的父级。实际上并不是一个设置。总是为<code>false</code>。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.obf</code></td>
      <td valign="top">在必要时强制指定refmap的混淆类型。总是为<code>false</code>。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.disableRefMap</code></td>
      <td valign="top">在必要时禁用refmap。</code>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.remapRefMap</code></td>
      <td valign="top">你可能希望在运行时重新映射现有的refmap，而不是禁用refmap。这可以通过设置此属性，并为<code>mixin.env.refMapRemappingFile</code>和<code>mixin.env.refMapRemappingEnv</code>属性设置值实现。不过，如果通过<code>GradleStart</code>启动，则可以忽略这些属性（如果通过GradleStart加载，此属性也会自动启用）。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.refMapRemappingFile</code></td>
      <td valign="top">如果启用<tt>mixin.env.remapRefMap</tt>，则可以使用此设置覆盖从文件中读取的SRG文件名。映射的源类型必须为<code>searge</code>且目标类型必须与当前的开发环境相匹配。 如果源类型不是<code>searge</code>，那么<code>mixin.env.refMapRemappingEnv</code>应设置为正确的源环境类型。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.refMapRemappingEnv</code></td>
      <td valign="top">使用<code>mixin.env.refMapRemappingFile</code>时，此设置将覆盖默认源环境（searge）。但是请注意，指定的环境类型必须存在于原始refmap中。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.ignoreRequired</code></td>
      <td valign="top">全局忽略所有配置的<code>required</code>属性。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.compatLevel</code></td>
      <td valign="top">操作时的默认兼容性级别。</td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.env.shiftByViolation</code></td>
      <td valign="top">在Mixin中超过<a href="http://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/injection/At.html#by--">At.by</a>所定义的最大值时的行为。当前行为是<code>warn</code>. 在Mixin的更高版本中，这可能会升级为<code>error</code>.
        <p>此选项的可用值包括：</p>
        <dl>
          <dt>ignore</dt>
          <dd>Pre-0.7行为，遇到违规行为时不采取任何操作</dd>
          <dt>warn</dt>
          <dd>当前行为，针对违规行为会发出一条<tt>WARN</tt>级别的消息</dd>
          <dt>error</dt>
          <dd>违规行为会抛出异常</dd>
        </dl>
      </td>
    </tr>
    <tr>
      <td valign="top"><code>mixin.initialiserInjectionMode</code></td>
      <td valign="top">初始化注入的行为，当前支持的选项有<code>default</code>和<code>safe</code>。</td>
    </tr>
  </tbody>
</table>

