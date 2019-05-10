---
title: 初试JNI
date: 2019-05-10 15:00:38
tags: [Java, C/C++]
---

> 开发环境：Java 11; IntelliJ IDEA 2019.1.1; Visual Studio 2019

**第一步：编写Java类**

```java
public class LearnJNI {

    public native void helloJNI();

    public static void main(String[] args) {
        System.loadLibrary("LearnJNI");
        new LearnJNI().helloJNI();
    }

}
```

**第二步：使用Java类生成C头文件**

在Java的编译输出目录下执行如下指令：
```
javah -classpath . -jni com.github.mouse0w0.learnjni.LearnJNI
```

结果如下图所示：

![Javah指令执行结果](1.png)

**第三步：使用Visual Studio创建C++项目**

在Visual Studio中选择C++空项目：

![创建新项目](2.png)

配置好项目名称，任取即可：

![配置新项目](3.png)

得到这样一个文件结构的项目：

![文件层次结构](4.png)

创建头文件与CPP文件，并将我们使用javah生成的头文件的内容复制到创建的头文件中：

![头文件](5.png)

我们可以看到有很多错误。不过接下来将配置我们的C++项目。

**第四步：配置C++项目**

右键解决方案，打开属性：

![解决方案属性](6.png)

选中`配置属性>配置`，设置配置为`Release`，平台为`x64`：

![设置配置属性](7.png)

右键`LearnJNI`项目，打开属性：

![项目属性](8.png)

选中`配置属性>常规`，设置`项目默认值>配置类型`为`动态库(.dll)`：

![设置配置类型](9.png)

选中`配置属性>VC++ 目录`，点击`常规>包含目录`的`<编辑...>`：

![编辑包含目录](10.png)

点击红框内按钮，添加JDK路径下的`include`文件夹和`include\win32`文件夹：

![包含目录](11.png)

**第五步：实现C++代码**

在CPP文件中编写如下代码，输出`Hello World`：

![CPP代码](12.png)

**第六步：编译C++代码为DLL**

点击Visual Studio上方的菜单栏中的`生成>生成 LearnJNI`生成项目：

![生成DLL](13.png)

最终得到如下输出，输出DLL的路径在下图红框内：

![生成DLL输出](14.png)

**第七步：配置Java运行**

将输出的DLL置于Java项目下的适宜文件夹：

![Native文件夹](15.png)

在IDEA的启动配置中增加JVM参数：

```
-Djava.library.path=D:\Workspace\JavaProjects\LearnJNI\native
```

![JVM参数](16.png)

运行启动配置，得到`Hello World`的输出：

![Hello World](17.png)

## 后记
本篇教程中所讲解的JNI和C/C++项目的开发和使用方法还很幼稚，读者可就情况进行进一步的学习和改进。如使用头文件自动生成工具，使用CMake配置跨平台的C/C++项目编译。

## 参考资料
[JNI 入门教程 | 菜鸟教程](https://www.runoob.com/w3cnote/jni-getting-started-tutorials.html)

[Visual Studio中开发Jni dll库](https://blog.csdn.net/l460133921/article/details/73824985)