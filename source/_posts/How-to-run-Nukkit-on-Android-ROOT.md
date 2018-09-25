---
title: 'How to run Nukkit on Android [ROOT]'
date: 2018-09-25 15:09:07
tags: [Android, Linux, Nukkit]
---
本文翻译自@Doomhawk的[教程](http://forums.voxelwind.com/threads/zapusk-servera-nukkit-na-android-root.118/)。文中部分图片为俄语，但我希望这篇教程会有所帮助，随便啦。

一个最常问的问题是“如何在Android上运行Nukkit？”。所以我想，何不写个教程？

首先，我必须要说的是，这并不简单。安卓不支持JDK或JRE，所以我们将安装Linux。

我们需要：
1. Android 4 及其以上版本。
2. 有ROOT权限的设备。
3. 2Gb的内部储存（不是SD卡）。
4. 1Gb以上的运行内存。
5. 一双可以打字的手。

开始吧！

1\. 从Google Play下载如下应用：

1.1. Terminal Emulator 

[Download from GooglePlay](https://play.google.com/store/apps/details?id=jackpal.androidterm&hl=ru)

![Terminal Emulator](te.png)

1.2. Complete Linux Installer

[Download from GooglePlay](https://play.google.com/store/apps/details?id=com.zpwebsites.linuxonandroid)

![Complete Linux Installer](cli.png)

1.3. ZArchiver

[Download from GooglePlay](https://play.google.com/store/apps/details?id=ru.zdevs.zarchiver)

![ZArchiver](za.png)

2\. 运行Complete Linux Installer。在首次运行时，它会请求Root权限以安装Busybox和一些必须的脚本：

![](ins.png)

同意。安装（大概15秒）后你必须关闭Complete Linux Installer。

3\. 你现在需要下载Linux，例如Ubuntu 14.04：

3.1. 单击链接并等待下载完成。

3.2. 运行ZArchiver，在你的SD卡的根目录下创建名为`ubuntu`的文件夹：

![](Ubu.png)

4.1. 现在打开你的下载目录。

4.2. 找到文件“ubuntu-14.04.CORE.ext4.PREALPHAv1.zip”，点击它。你会看到以下菜单：

![](tap.png)

4.3. 点击“Extract”，打开文件夹“ubuntu”，然后点击图标：

![](ext.png)

4.4. 等待直到文件被解压。

4.5. 将文件“ubuntu-14.04.CORE.ext4.img”重命名为“ubuntu.img”，并将文件“ubuntu-14.04.CORE.ext4.img.md5”也改为“ubuntu.img.md5”：

![](ren.png)

现在你可以卸载ZArchiver了，我们再也不需要它了。

5\. 再次运行Complete Linux Installer，打开菜单（右边）：

![](swipe.png)

5.1. 选择“Run”然后选择“Ubuntu”。

![](sel.png)

5.2. 按下“Run Linux”然后你将看到正在运行的“Terminal Emulator”：

![](tm.png)

5.3. 在首次运行时，你会收到一些提示。“MD5 file found, use to check .img file?”，输入“n”（否）。

5.4. 在此步骤中，我收到了错误"can't execute '/root/init.sh': Permission denied"：

![](err.png)

我不确定所有人都会有相同的错误。但这是我的解决方案：

运行指令:
```
mv /data/data/com.zpwebsites.linuxonandroid/files/busybox /data/data/com.zpwebsites.linuxonandroid/files/busybox.bak
```

![](ans.png)

然后按下“Windows 1”，点击X号。终端将会被关闭，再次运行Complete Linux Installer并点击“Run Linux”。

耶！它正常工作啦！

![](noerr.png)

5.5. 现在，你被要求输入密码。（输入新的UNIX密码）请输入你的密码。

警告！在输入时，你将看不到密码（起点或者是点）。

5.6. 重复密码。

5.7. 看到提示“Start VNC server?”和“Start SSH server?”输入“n”（否），看到提示"Save settings as defaults?"输入“y”（是）。

5.8. 太好了！Linux已经安装完成，你肯定看到`root@localhost!:`显示在你的屏幕上了。

![](okk.png)

6.1. 现在你需要安装Java 8和Nukkit。在控制台执行如下指令：
```
apt-get update
apt-get install software-properties-common python-software-properties -y
add-apt-repository ppa:openjdk-r/ppa -y
apt-get update
apt-get install openjdk-8-jdk openjdk-8-jre -y
wget http://ci.mengcraft.com:8080/job/nukkit/lastSuccessfulBuild/artifact/target/nukkit-1.0-SNAPSHOT.jar
java -jar nukkit-1.0-SNAPSHOT.jar
```

你现在肯定看到Nukkit成功运行啦！

![](run.png)

然后你可以使用MCPE尝试登陆。

![](join.png)

当你要关闭Linux时，你可以在控制台中输入“exit”，或者下次你必须重复此前所有步骤。