---
title: 使用VSCode扩展REST Client发送HTTP请求
date: 2018-10-10 22:54:51
tags: [Tool, Web]
---

最近在参加[中国科学技术大学第五届信息安全大赛](https://hack.lug.ustc.edu.cn/)时，经高人指点，本渣找到了一款比Postman要方便（个人认为）的发送HTTP请求的工具，就是VSCode扩展[REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)，特此写博客记述一下。

## 安装
非常简单，在VSCode按`F1`，输入`ext install`后搜索`rest-client`即可。

## 简单GET
```
GET https://example.com/comments/1 HTTP/1.1
```

然后使用快捷键`Ctrl+Alt+R`即可运行，结果将会显示在右栏。

## 后记
REST Client还有Java应用程序版本，Github地址在此：[https://github.com/wiztools/rest-client](https://github.com/wiztools/rest-client)。

不多说其实它还有更多实现，不过就在此不做更多介绍了。