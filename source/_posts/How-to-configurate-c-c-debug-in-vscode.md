---
title: 如何在VSCode中配置C/C++ Debug
date: 2018-11-03 20:34:10
tags: [VSCode, C, C++]
---

## 下载并安装MinGW
首先进入[MinGW官网](http://www.mingw.org/)，点击Download按钮。

![MinGW Download](MinGW_Download_1.png)

进入下载页面后，点击**MinGW Installation Manager (mingw-get)**。

![MinGW Download](MinGW_Download_2.png)

进入mingw-get的下载页面后，点击**Show**按钮，然后分别下载**mingw-get-0.6.3-mingw32-pre-20170905-1-bin.tar.xz**和**mingw-get-0.6.3-mingw32-pre-20170905-1-gui.tar.xz**文件（此过程可能需要挂VPN）。

![MinGW Download](MinGW_Download_3.png)

> 注意：其实直接用`mingw-get-setup.exe`安装MinGW也可以，略去了解压步骤。

两个文件下载完成后，将其解压至同一文件夹中。

![MinGW Unzip](MinGW_Unzip.png)

进入MinGW目录下的`libexec\mingw-get`文件夹，点击`guimain.exe`运行。

![MinGW GUI](MinGW_GUI.png)

等待MinGW载入完成，这个过程可能很长。

![MinGW Loading](MinGW_Loading.png)

加载完成后看起来如图所示，其中**Basic Setup**是基本要安装的，**All Packages**则包含了所有可以安装的包，具体各包的作用我们在此不做深入探讨了。

![MinGW Loaded](MinGW_Loaded.png)

**右键单击**想要安装的项目，选择**Mark for Installation**对想要安装的项目进行标记，同时该项目的依赖项也会被标记为需要安装。

![MinGW Mark for Installation](MinGW_Mark.png)

在**All Packages**中找到`mingw32-gcc-g++-bin`和`mingw32-gdb-bin`，标记它们为需要安装的项目，此时它们的依赖项也会被同时标记。

![MinGW Mark for Installation](MinGW_Mark_2.png)

点击左上角菜单栏的**Installation**，选择**Apply Changes**，在弹出的界面中再点击**Apply**开始安装所选的项目。

![MinGW Installation](MinGW_Installation.png)

![MinGW Installation](MinGW_Installation_2.png)

等待它安装完成，这个过程可能会很长。安装完成后点击**Close**按钮即可。

![MinGW Installing](MinGW_Installing.png)

在**计算机-系统属性-高级系统设置-环境变量**将`MinGW\bin`添加到`Path`中。

![MinGW Environment Variable](MinGW_Env_Var.png)

打开`cmd`，输入`gcc -v`，输出如下信息即表示安装成功。

![MinGW Command Line](MinGW_Cmd.png)

## 配置VSCode

在扩展中搜索C，然后选择`C/C++`插件进行安装。

![VSCode Install C](VSCode_Install_C.png)

安装完成后重启，并打开一个文件夹作为C/C++开发的工作空间。

点击上方菜单栏中的**调试-打开配置**。

![VSCode Configurate](VSCode_Configurate_1.png)

在打开的`launch.json`文件里面，添加如下内容：
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，这里只能为cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${workspaceRoot}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，一般设置为false
            "cwd": "${workspaceRoot}", // 调试程序时的工作目录，一般为${workspaceRoot}即代码所在目录
            "environment": [],
            "externalConsole": true, // 调试时是否显示外部控制台窗口
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\MinGW\\gdb.exe", // miDebugger的路径，注意这里要与MinGw的路径对应
            "preLaunchTask": "g++", // 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```
结果如下所示：

![VSCode Configurate](VSCode_Configurate_2.png)

然后保存`launch.json`文件。

然后点击上方菜单栏中的**终端-配置任务...**。然后选择**使用模板创建 tasks.json 文件**，再选择**Others 运行任意外部命令的示例**，这将打开一个`tasks.json`文件。

![VSCode Configurate](VSCode_Configurate_3.png)

![VSCode Configurate](VSCode_Configurate_4.png)

![VSCode Configurate](VSCode_Configurate_5.png)

在打开的`tasks.json`文件中写入以下内容：
```json
{
    "version": "2.0.0",
    "command": "g++",
    "args": [
        "-g",
        "${file}",
        "-o",
        "${fileBasenameNoExtension}.exe",
        "-fexec-charset=GBK"
    ], // 编译命令参数
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": [
            "relative",
            "${workspaceRoot}"
        ],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    }
}
```
结果如下所示：

![VSCode Configurate](VSCode_Configurate_6.png)

接下来，我们编写一个HelloWorld程序，然后测试它：

![Hello World](VSCode_HelloWorld.png)

点击上方菜单栏的**调试-非调试启动**以开始运行：

![VSCode Run](VSCode_Run.png)

最终结果如下所示：

![VSCode Successful](VSCode_Successful.png)
