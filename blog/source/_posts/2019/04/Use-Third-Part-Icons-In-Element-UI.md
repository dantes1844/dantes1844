---
title: （转）Vue Element使用icon图标(第三方)
date: 2019-04-20 19:39:21
tags: [VUE,javascript]
categories: 
	- [前端]
---

找不到文章的转载按钮，将连接附在下方吧。
[Vue Element使用icon图标(第三方)](https://www.jianshu.com/p/59dd28f0b9c9)

我使用的是cdn版的vue和element-UI。过程稍微有一点曲折。



有两个注意的地方：
1：自定义icon的css要放在element-UI的css之后，这样才能正确解析出来。

2：下载的css的font-size是16px，会跟现有的不一致，将button撑大，改成12px就好了。

另外这位兄弟添加到购物车的操作确实很“程序员”，这个要多学习下。