---
title: 【转载】解决Web部署 svg/woff/woff2字体 404错误
date: 2019-06-11 17:23:32
tags: [IIS]
---

转载：https://www.cnblogs.com/hanwen/p/4212622.html

问题：最近在IIS上部署web项目的时候，发现浏览器总是报找不到woff、woff2字体的错误。导致浏览器加载字体报404错误，白白消耗了100-200毫秒的加载时间。

原因：因为服务器IIS不认SVG，WOFF/WOFF2 这几个文件类型，只要在IIS上添加MIME 类型即可。

解决方法
1、打开服务器IIS管理器，找到MIME类型。
![image.png](https://upload-images.jianshu.io/upload_images/2665968-01e0fc8ba9e6870a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、添加MIME类型 添加三条：　　

       文件扩展名      MIME类型　

.svg             image/svg+xml
.woff            application/x-font-woff
.woff2          application/x-font-woff