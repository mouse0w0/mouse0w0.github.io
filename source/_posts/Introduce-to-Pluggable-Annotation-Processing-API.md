---
title: 插入式注解处理API（JSR269）介绍
date: 2018-11-28 00:00:00
tags: [Java]
---
插入式注解处理API（[JSR269](https://www.jcp.org/en/jsr/detail?id=269)）是用于处理注解（元数据，[JSR175](https://www.jcp.org/en/jsr/detail?id=269)）的一套API。其API位于`javax.annotation.processing`和`javax.lang.model`包下。

插入式注解处理API可以让你在编译期访问注解元数据，处理和自定义你的编译输出，像反射一样访问类、字段、方法和注解等元素，创建新的源文件等等。可用于减少编写配置文件的劳动量，提高代码可读性等等。本文中我们将尝试着创建一个用于标识服务提供者的注解处理器，并同时讲解插入式注解处理API的相关内容。

> 关于服务提供者和`ServiceLoader`是什么及其更多信息请浏览Java文档[java.util.ServiceLoader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)

首先创建一个`@ServiceProvider`注解，该注解用于标识一个服务提供者类，其`value()`值为服务接口类的类对象。
```java
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface ServiceProvider {
    Class<?> value();
}
```

接下来，我们为该注解创建一个注解处理器，注解处理器需实现`javax.annotation.processing.Processor`接口或继承`javax.annotation.processing.AbstractProcessor`类。
```java
public class ServiceProviderProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        return false;
    }
}
```

同时，我们要在我们的源文件路径下的`META-INF/services`文件夹下，创建一个名为`javax.annotation.processing.Processor`的文本文件，其中写入我们的注解处理器的全称，例如：
```
com.github.mouse0w0.jsr269.ServiceProviderProcessor
```

在services下添加了注解处理器服务后，Java编译器就能够自动地发现你的注解处理器了。

现在的文件结构如下所示：
```
src
|- com
    |- github
        |- mouse0w0
            |- jsr269
                |- ServiceProvider.java
                |- ServiceProviderProcessor.java
|- META-INF
    |- services
        |- javax.annotation.processing.Processor
pom.xml
```

为了能够让我们的项目能够通过编译，我们需要为Java编译器添加一个不进行注解处理的参数。假设我们使用Maven构建项目，那么在`pom.xml`文件中添加编译参数设置：
```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <compilerArgument>-proc:none</compilerArgument>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
这样，Java编译器就不会试图寻找我们的注解处理器了。

接下来介绍一下`ProcessingEnvironment`接口。该接口表示一个处理环境，可以通过实现`init(ProcessingEnvironment)`方法或使用`processingEnv`字段取得。它提供了许多接口用于与编译器、源文件和类文件的交互。有了它，你就可以发送编译信息（提示、警告甚至是错误等等），创建新的源文件或类文件。它的接口声明如下：
```java
public interface ProcessingEnvironment {

    Map<String,String> getOptions();

    /**
     * 返回消息发送器，用于发送消息
     */
    Messager getMessager();

    /**
     * 返回文件访问器，用于创建源文件、类文件或者其他文件
     */
    Filer getFiler();

    Elements getElementUtils();

    Types getTypeUtils();

    /**
     * 返回当前的源代码版本
     */
    SourceVersion getSourceVersion();

    /**
     * 返回当前的语言环境
     */
    Locale getLocale();
}
```

接下来我们开始着手编写我们的注解处理器吧。首先我们需要重写两个方法：
```java
   @Override
    public SourceVersion getSupportedSourceVersion() { // 支持的源代码版本
        return SourceVersion.RELEASE_8;
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() { // 支持的注解类型
        Set<String> set = new HashSet<>();
        set.add(ServiceProvider.class.getName());
        return set;
    }
```

然后，向`process`方法中添加我们的代码逻辑：
```java
    private final Map<String, List<String>> services = new HashMap<>();

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        if (!roundEnv.processingOver()) { // 判断是否为最终轮
            for (Element element : roundEnv.getElementsAnnotatedWith(ServiceProvider.class)) {
                if (!(element instanceof TypeElement)) {
                    continue;
                }
                addService(findAnnoValue(element, ServiceProvider.class.getName(), "value").getValue().toString(), ((TypeElement) element).getQualifiedName().toString());
            }
        } else {
            saveAll();
        }
        return false; // 如果为true，则接下来的处理器不可处理该注解；如果为false，则接下来的处理器可以处理该处理器处理的注解。
    }

    private AnnotationValue findAnnoValue(Element element, String annoType, String key) {
        for (AnnotationMirror anno : element.getAnnotationMirrors()) {
            if (!ServiceProvider.class.getName().equals(anno.getAnnotationType().toString())) {
                continue;
            }
            for (Map.Entry<? extends ExecutableElement, ? extends AnnotationValue> entry : anno.getElementValues().entrySet()) {
                if (key.equals(entry.getKey().getSimpleName().toString())) {
                    return entry.getValue();
                }
            }
        }
        return null;
    }

    private void addService(String service, String provider) {
        services.computeIfAbsent(service, s -> new ArrayList<>()).add(provider);
    }

    private void saveAll() {
        for (Map.Entry<String, List<String>> entry : services.entrySet()) {
            FileObject fileObject = null;
            try {
                fileObject = processingEnv.getFiler().createResource(StandardLocation.CLASS_OUTPUT, "", "META-INF/services/" + entry.getKey());
            } catch (IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, e.getMessage());
                e.printStackTrace();
            }

            if (fileObject == null) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Cannot create file object for " + entry.getKey());
                return;
            }

            try (Writer writer = fileObject.openWriter()) {
                for (String s : entry.getValue()) {
                    writer.append(s).append("\n");
                }
            } catch (IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, e.getMessage());
                e.printStackTrace();
            }
        }
    }
```

现在，你可以将注解处理器打包为Jar类库，然后由你的其他项目引用并构建。就会自动地在`META-INF/services`下生成对应的服务提供者文件了！
```java
@ServiceProvider(MyService.class)
public class MyServiceImpl implements MyService {
}
```

最后，使用`ServiceLoader.load(MyService.class)`加载对应服务类的所有服务提供者实例：
```java
for (MyService service : ServiceLoader.load(MyService.class)) {
    System.out.println(service.getClass().getSimpleName()); // 输出为MyServiceImpl
}
```

更多关于插入式注解处理API的详细信息，请浏览Java文档[javax.annotation.processing](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html)和[javax.lang.model](https://docs.oracle.com/javase/8/docs/api/javax/lang/model/package-summary.html)。

> 强烈建议使用Maven或Gradle等自动构建工具进行构建，否则你可能需要配置你的IDE以使用注解处理器。

> 后记：我实在是懒得写了......