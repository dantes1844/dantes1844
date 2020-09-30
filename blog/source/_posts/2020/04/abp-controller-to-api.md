---
title: 【ABP框架笔记】 5.控制器自动生成api接口的事项
permalink: abp-controller-to-api
date: 2020-04-30 16:40:58
tags: [ABP,.net core 3.0,ef core 3.0]
categories:   
    - [框架]
    - [ABP]
---



在研究`RemoteServiceAttribute`时，在控制器中使用它的时候，发现以下几个情况



#### 1.  RemoteServiceAttribute使用方式

注意：必须在`Controller`和`Action`上**同时**打上该标签，才会生成对应的`api`接口。

如果一个`Action`上既有`RemoteServiceAttribute`又有`ApiExplorerSettings`那么`abp`不会处理，会将其交由`.net core`本身去处理。



#### 2. 控制器要生成api接口应当使用的配置

##### 2.1 特性标签

```c#
[Route("api/Custom")]
[ApiExplorerSettings(IgnoreApi = false)]
```

这两个必须同时出现，才能将一个控制器转成`api`接口。单独将控制器启用`ApiExplorerSettings`会报错，异常信息：

>  InvalidOperationException: The action   'AbpAspNetCoreDemo.Controllers.ProductsController.UglyActionNameForSearch (AbpAspNetCoreDemo)' has ApiExplorer enabled, but is using conventional routing. Only actions which use attribute routing support ApiExplorer.

##### 2.2 应当在每个方法(action)而不是控制器上打标签

在控制器类上打标签后，生成的`api`地址都是一样的，且方法都是`POST`.要在每个`Action`单独设置请求地址。

要想将`Action`转成`Get`请求，需要在`Action`上打标签`[HttpGet]`



##### 2.3 相同名称的api地址

在服务类里生成了 `api/services/app/Product/GetAll`的请求地址，然后再在控制里添加`Product`控制器和`GetAll`方法，`js`里会有两份一模一样的`api`方法

```
// action 'getProducts' 这里是服务类生成的
    abp.services.app.product.getProducts = function(ajaxParams) {
      return abp.ajax($.extend(true, {
        url: abp.appPath + 'api/services/app/product/GetAll',
        type: 'GET'
      }, ajaxParams));;
    };

    // action 'getAll' 这里是控制器生成的
    abp.services.app.product.getAll = function(ajaxParams) {
      return abp.ajax($.extend(true, {
        url: abp.appPath + 'api/services/app/Product/GetAll',
        type: 'GET'
      }, ajaxParams));;
    };
```



在发送请求时，后端会报错，提示有歧义的请求。所以在添加服务类或者控制器的时候，要避免相同路径出现。

![相同路径的api报错](AmbiguousMatchException.png)