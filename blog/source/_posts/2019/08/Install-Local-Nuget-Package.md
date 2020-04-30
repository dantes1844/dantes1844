---
title: 安装离线nuget包
date: 2019-08-19 08:56:09
tags: [Visual Studio,Nuget]
categories: 
	- [工具]
---

Visual Studio 2013 的nuget包管理突然就不能使用了，周边的同事都试了，问题应该是公司的网络故障。

在 [nuget 官网](https://www.nuget.org/) 的包下载页面可以看到有离线包下载的功能，于是找了相关资料，记录下使用离线 `nuget` 包的使用方法.

参考连接：[Stackoverflow](https://stackoverflow.com/questions/10240029/how-do-i-install-a-nuget-package-nupkg-file-locally)

1. 下载离线包

![官网包页面找到下载连接](1.png)

2. 下载的包在本地文件存储下来。

![存储文件到本地](2.png)

3. Visual Studio 增加包下载的地址

![Visual Studio相关设置](3.png)

4. 安装包

![安装](4.png)

需要注意的是：如果当前包依赖于其他的包，那么`nuget`管理器会首先去检查本地的路径有没有相关的包，有就使用本地的进行安装，否则还是会连接到`nuget`服务器去下载，所以在网络不通的情况下，依赖包也是需要提前下载好的。否则当前包依然会安装失败。
