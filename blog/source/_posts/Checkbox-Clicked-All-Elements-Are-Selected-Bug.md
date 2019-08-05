---
title: element UI CheckBox单击操作时一组的元素都被选中的问题
date: 2019-04-11 20:40:55
tags: [element-UI,Vue,javascript]
---

在重构项目时，有个功能是一组CheckBox的勾选功能，按官方实例：[http://element-cn.eleme.io/#/zh-CN/component/checkbox](http://element-cn.eleme.io/#/zh-CN/component/checkbox)，绑定上后台返回的数据时，在单击任何一个CheckBox时，这一组都被选中了，后来经过排查发现，原来后台返回的绑定数据是一个null值，将后台返回的数据改成空数组之后，一切恢复正常。






![1.gif](https://upload-images.jianshu.io/upload_images/2665968-c7e9627459dcd11c.gif?imageMogr2/auto-orient/strip)
