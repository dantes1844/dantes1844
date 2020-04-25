---
title: ninject +entity framework+控制器过滤器 dbcontext被释放的bug
date: 2019-06-06 17:28:14
tags: [Entity Framework,Ninject,过滤器]
categories：
	- [框架]
---

### 问题描述

> entity framework数据层，使用ninject进行注入，生命周期设置为InRequestScope，在过滤器中调用数据库时，会出现` The operation cannot be completed because the DbContext has been disposed.`

![image.png](https://upload-images.jianshu.io/upload_images/2665968-7a47deefa11b5919.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 原因分析

在SO上找到几个相关的问题，如下：

1. [https://stackoverflow.com/a/6194159/2031686](https://stackoverflow.com/a/6194159/2031686)

2. [https://stackoverflow.com/questions/6193414/dependency-injection-with-ninject-and-filter-attribute-for-asp-net-mvc/27011327](https://stackoverflow.com/questions/6193414/dependency-injection-with-ninject-and-filter-attribute-for-asp-net-mvc/27011327)

3. [https://github.com/ninject/Ninject.Web.Mvc/wiki/Dependency-injection-for-filters](https://github.com/ninject/Ninject.Web.Mvc/wiki/Dependency-injection-for-filters)

4. [https://stackoverflow.com/a/7834381/2031686](https://stackoverflow.com/a/7834381/2031686)

> Filter Providers aren't created for every request and therefore aren't allowed to get any dependency in request scope.

其中3里面github是官方关于过滤器中使用注入的方式介绍，前面两个答案也基本是采用了这个方式。

### 解决方法

最终采用了上面1里面的，BZ的回答。

步骤

1. 定义一个特性类，给要使用的控制器或者action打上相关标签

2. 定义一个过滤器，该过滤器里使用构造函数的方式获取注入的实体对象并使用。我这里就是调用数据库的DbContext

3. 使用如下方式进行绑定

	```
	this.BindFilter<MyAuthorizeFilter>(System.Web.Mvc.FilterScope.Controller, 0).WhenControllerHas<MyAuthorizeAttribute>();
	```
