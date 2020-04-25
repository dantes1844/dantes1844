---
title: 【ABP框架笔记】2.ApplicationServices 生成web api
date: 2020-04-25 14:39:05
tags: [ABP,.net core 3.0,ef core 3.0]
categories: 
	- [框架]
	- [Web]
---

### 一、Laobai.Application 业务层的配置

本层中会有一个`LaobaiApplicationModule`，该类继承自`AbpModule`，在`Laobai.Web.Core`中将使用该类作为配置的起点。

本层中放置全部的业务类，默认都继承自`AsyncCrudAppService`，`AsyncCrudAppService`以虚方法默认实现了几个通用的全局方法，如下：

1. GetAsync
2. GetAllAsync
3. CreateAsync
4. UpdateAsync
5. DeleteAsync
6. GetEntityByIdAsync

以上方法会在默认的`api`中出现，在实现的时候，会将`Async`从方法名中去掉，最终`api`的方法类似于`GetAll`等。

### 二. 全局配置service转为api

[官方文档](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core) 

在`Laobai.Web.Core`项目里的`LaobaiWebCoreModule`中，有如下配置代码：

```
public override void PreInitialize()
        {
            ...

            //配置动态web api 
            Configuration.Modules.AbpAspNetCore()
                 .CreateControllersForAppServices(
                     typeof(LaobaiApplicationModule).GetAssembly(), "laobai"
                 );
                 
			...
        }
```

`CreateControllersForAppServices` 方法有三个参数：

1. 第一个参数是要转化为webapi的服务类所在的程序集。

2. 第二个参数是要生成的webapi中的模块名称。默认使用的是`app`这个名字。

   >  猜测：这里可以采用不同的模块名称，将不同的服务放到不同的webapi分类中。不过这样可能需要分程序集了，待验证？

3. 第三个参数是根据方法名称使用约定的`http`请求动作，例如`Get`，`Post`，`Put`，`Delete`等，给的的默认值是`true`也就是要按照方法名生成不同的请求动作，如果是`false`，那么所有的动作默认为`post`请求。具体的实现代码:

   ``````c#
   var verb = configuration?.UseConventionalHttpVerbs == true
                              ? ProxyScriptingHelper.GetConventionalVerbForMethodName(action.ActionName)
                              : ProxyScriptingHelper.DefaultHttpVerb;
   ``````

   ```c#
   public static string GetConventionalVerbForMethodName(string methodName)
   {
       if (methodName.StartsWith("Get", StringComparison.OrdinalIgnoreCase))
       {
           return "GET";
       }
   
       if (methodName.StartsWith("Put", StringComparison.OrdinalIgnoreCase) ||
           methodName.StartsWith("Update", StringComparison.OrdinalIgnoreCase))
       {
           return "PUT";
       }
   
       if (methodName.StartsWith("Delete", StringComparison.OrdinalIgnoreCase) ||
           methodName.StartsWith("Remove", StringComparison.OrdinalIgnoreCase))
       {
           return "DELETE";
       }
   
       if (methodName.StartsWith("Patch", StringComparison.OrdinalIgnoreCase))
       {
           return "PATCH";
       }
   
       if (methodName.StartsWith("Post", StringComparison.OrdinalIgnoreCase) ||
           methodName.StartsWith("Create", StringComparison.OrdinalIgnoreCase) ||
           methodName.StartsWith("Insert", StringComparison.OrdinalIgnoreCase))
       {
           return "POST";
       }
   
       //Default
       return DefaultHttpVerb;
   }
   ```

   

   

   生成的`api`默认路径是 **/api/services/<第二个参数，默认是app>/<业务类名称>/<方法名称>**



### 三.RemoteServiceAttribute 禁用服务类转成api接口

`RemoteServiceAttribute`特性的适用范围：类，方法，接口。

需要将该特性类的构造参数设置为`false `才是禁用，默认是`true`，也就是不禁用。

- 指定的方法上打上该标签，那么该方法将被忽略生成`api`资源。

- 指定的类上打上该标签，那么类下面的所有方法都不会生成`api`资源，包括该类的父类`AsyncCrudAppService`下的几个默认的方法。

- ***注意：指定的接口上打上该标签，不起作用。***

### 四、 使用HttpMethodAttribute设置资源动作

`HttpMethodAttribute`包括以下几种标签。

- HttpGet
- HttpPut
- HttpDelete
- HttpHead
- HttpPost
- HttpOptions
- HttpPatch

如果设置了`UseConventionalHttpVerbs`，那么会根据方法的开头来确定动作的类型， 如果方法的开头不包含默认的几种，则使用`Post`。

```c#
[HttpGet]
public async Task<ListResultDto<RoleDto>> LaobaiGetRoles()
{
    var roles = await _roleRepository.GetAllListAsync();
    return new ListResultDto<RoleDto>(ObjectMapper.Map<List<RoleDto>>(roles));
}
```

以上示例代码使用`HttpGetAttribute`，可以指定非特定字符开头的方法为指定的资源动作。有两个事项：

1. 这个时候，客户端就能以`Get`方法获取`LaobaiGetRoles`这个资源了。
2. `HttpGet`的构造函数必须使用无参数的那个，否则会根据路由规则生成新的`api`地址，例如本例中就变成了 `/GetRoles`，没有与其他`api`一样的默认的前缀路径，客户端就会找不到资源。



