---
title: 用Kotlin开发Forge模组
date: 2018-08-24 20:19:00
tags: Forge
---

首先打开ForgeMdk目录下的`build.gradle`文件，在其中添加仓库：
```gradle
repositories {
    mavenCentral()
    jcenter()
    maven {
        url "http://maven.shadowfacts.net/
    }
}
```
然后为模组添加依赖：
```gradle
dependencies {
    compile "org.jetbrain.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    compile "net.shadowfacts:Forgelin:1.6.0"
    // ...
}
```
设置所使用的Kotlin版本：
```gradle
ext.kotlin_version = '1.2.10'
```
并添加使用对应的Kotlin Gradle插件：
```gradle
apply plugin: 'net.minecraftforge.gradle.forge'
apply plugin: 'kotlin'
```
最后在你的`@Mod`注解中添加以下内容：
```java
@Mod(modLanguageAdapter="net.shadowfacts.forgelin.KotlinAdapter")
```
现在只需要点击你IDE上的Gradle项目刷新按钮就可以使用Kotlin开发Forge模组啦！

参考文献：[http://couchdoescode.blogspot.com/2018/01/minecraft-modding-tutorial-with-kotlin.html](http://couchdoescode.blogspot.com/2018/01/minecraft-modding-tutorial-with-kotlin.html)

Forgelin项目地址：[GitHub](https://github.com/shadowfacts/Forgelin)