---
title: 介绍PlantUML
date: 2020-06-25 21:30:00
tags: [Tool, Design Pattern]
---

为了描述项目的编码需求，常常需要使用一些辅助制图工具，UML图就是一个不错的选择。在经过不断地试用和实践过程中，最终决定使用PlantUML。PlantUML是一个可以使用简单的语法描述UML图的开源工具，也可以简单有效的定制UML图的样式。

PlantUML在VSCode，IntelliJ Idea，Eclipse等多款软件中均有插件支持，下图仅演示如何安装VSCode的PlantUML扩展。

![安装 PlantUML 扩展](vscode_extension.png)

然后，要使用PlantUML的全部功能，还需要安装**Graphviz**。Graphviz是一个开源的图形可视化软件。下载地址：[http://www.graphviz.org/download/](http://www.graphviz.org/download/)

安装完毕后就可以在VSCode中编写你想要的UML类图了。

`example.puml`：
```
@startuml
Bob->Alice : hello
@enduml
```

![PlantUML 示例](example.png)

> 注：按快捷键`Alt+D`即可预览PlantUML结果，右键菜单中有更多功能可供选择。

关于PlantUML图的语法，在此不做赘述，请前往官网查看相关教程：[PlantUML 中文官网](https://plantuml.com/zh/)

PlantUML的默认配色不一定很适合所有人使用，有些人更喜欢简洁一些的主题，可以在`@startuml`后添加`skinparam monochrome true`参数。

`example.puml`：
```
@startuml
skinparam monochrome true
Bob->Alice : hello
@enduml
```

![Monochrome 主题](monochrome.png)

有的人更喜欢暗色主题，可以在`@startuml`后添加`skinparam monochrome reverse`参数使用暗色主题。

`example.puml`：
```
@startuml
skinparam monochrome reverse
Bob->Alice : hello
@enduml
```

![Monochrome 反转主题](reverse_monochrome.png)

以上就是PlantUML提供的一些主题，如有需要可以自行阅读PlantUML的[用户指南](http://plantuml.com/zh/guide)，通过`skinparam`修改样式。

再推荐一个非官方主题，有需要的可自行取用：[https://github.com/xuanye/plantuml-style-c4](https://github.com/xuanye/plantuml-style-c4)。
