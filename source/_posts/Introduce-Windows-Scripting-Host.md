---
title: 介绍 Windows Scripting Host
date: 2018-10-13 14:08:31
tags: [Windows, Script]
---
近日为了使用脚本创建快捷方式，特地查阅了相关资料，最终寻找到了Windows下隐藏的实用功能。它就是**Windows Scripting Host**。

> WSH 是“Windows Scripting Host”的缩略形式，其通用的中文译名为“Windows脚本宿主”。对于这个较为抽象的名词，我们可以先作这样一个笼统的理解：它是内嵌于 Windows 操作系统中的脚本语言工作环境。

## 简介
Windows Scripting Host 这个概念最早出现于 Windows 98 操作系统。大家一定还记得 MS-Dos 下的批处理命令，它曾有效地简化了我们的工作、带给我们方便，这一点就有点类似于如今大行其道的脚本语言。但就算我们把批处理命令看成是一种脚本语言，那它也是 98 版之前的 Windows 操作系统所唯一支持的“脚本语言”。而此后随着各种真正的脚本语言不断出现，批处理命令显然就很是力不从心了。面临这一危机，微软在研发 Windows 98 时，为了实现多类脚本文件在 Windows 界面或 Dos 命令提示符下的直接运行，就在系统内植入了一个基于 32 位 Windows 平台、并独立于语言的脚本运行环境，并将其命名为“Windows Scripting Host”，WSH 架构于 ActiveX 之上，通过充当 ActiveX 的脚本引擎控制器，WSH 为 Windows 用户充分利用威力强大的脚本指令语言扫清了障碍。

再具体一点描述：你自己编写了一个脚本文件，如后缀为 .vbs 或 .js 的文件，然后在 Windows 下双击并执行它，这时，系统就会自动调用一个适当的程序来对它进行解释并执行，而这个程序，就是 Windows Scripting Host，程序执行文件名为 Wscript.exe （若是在命令行下，则为 Cscript.exe）。

## 相关文档
- [WScript](https://msdn.microsoft.com/en-us/library/at5ydy31)
- [WScript.Shell](https://msdn.microsoft.com/en-us/library/aew9yb99)

## .wsh 文件运行脚本
```toml
[ScriptFile]
Path=C:/YourScript.vbs

[Options]
Timeout=25
DisplayLogo=0
```

## 实战：创建桌面快捷方式
VBScript（.vbs)实现：

```vb
CurrentPath = WScript.CreateObject("Scripting.FileSystemObject").GetFolder(".").Path
SET WshShell = WScript.CreateObject("WScript.Shell")
Desktop = WshShell.SpecialFolders("Desktop")

SET oShellLink = WshShell.CreateShortcut(CurrentPath & "\Notepad.lnk")
oShellLink.TargetPath = "notepad.exe"
oShellLink.WindowStyle = 1
oShellLink.Description = "Notepad"
oShellLink.WorkingDirectory = Desktop
oShellLink.Save

CompletedMessage = "Completed"
WScript.echo CompletedMessage
```

JScript(.js)实现：

```js
var WshShell = WScript.CreateObject("WScript.Shell");
var Desktop = WshShell.SpecialFolders("Desktop");
var oShellLink = WshShell.CreateShortcut(Desktop + "\\Notepad.lnk")
oShellLink.TargetPath = "notepad.exe";
oShellLink.WindowStyle = 1;
oShellLink.Description = "Notepad";
oShellLink.WorkingDirectory = Desktop;
oShellLink.Save();

WScript.echo("Completed");
```

## .wsf 文件
此外，还可以使用.wsf文件，这是一个XML格式的文件。
```xml
<package>
   <job id="vbs">
      <script language="VBScript">
         set WshShell = WScript.CreateObject("WScript.Shell")
         strDesktop = WshShell.SpecialFolders("Desktop")
         set oShellLink = WshShell.CreateShortcut(strDesktop & "\Shortcut Script.lnk")
         oShellLink.TargetPath = WScript.ScriptFullName
         oShellLink.WindowStyle = 1
         oShellLink.Hotkey = "CTRL+SHIFT+F"
         oShellLink.IconLocation = "notepad.exe, 0"
         oShellLink.Description = "Shortcut Script"
         oShellLink.WorkingDirectory = strDesktop
         oShellLink.Save
      </script>
   </job>

   <job id="js">
      <script language="JScript">
         var WshShell = WScript.CreateObject("WScript.Shell");
         strDesktop = WshShell.SpecialFolders("Desktop");
         var oShellLink = WshShell.CreateShortcut(strDesktop + "\\Shortcut Script.lnk");
         oShellLink.TargetPath = WScript.ScriptFullName;
         oShellLink.WindowStyle = 1;
         oShellLink.Hotkey = "CTRL+SHIFT+F";
         oShellLink.IconLocation = "notepad.exe, 0";
         oShellLink.Description = "Shortcut Script";
         oShellLink.WorkingDirectory = strDesktop;
         oShellLink.Save();
      </script>
   </job>
</package>
```