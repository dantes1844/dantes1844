---
title: 【ABP框架笔记】 4.高频使用的Attributes
permalink: abp-attribute-list
date: 2020-04-30 09:10:40
tags: [ABP,.net core 3.0,ef core 3.0]
categories: 
    - [框架]
    - [ABP]
---



#### 1. NotNull,CanBeNull,ItemNotNull等

这些其实是ReSharper插件提供的，在`JetBrains.Annotations`命名空间下，它是`ReSharper`自己智能提示用的，例如：在参数上加上`[NotNull]`标签之后，在传入的参数为`null`时，`ReSharper`的智能提示会给出警告信息。但是这个提醒并不会影响编译。这个提示可以让开发人员在使用参数时，注意检查参数的可空问题。

![NotNull的参数提示](NotNullAttribute.png)

![检查参数为空](CheckNotNull.png)

可以在[ReSharper官网](https://www.jetbrains.com/help/resharper/Reference__Code_Annotation_Attributes.html)查看更多类似的`Attribute`

#### 2.  DebuggerStepThrough

这个在`ABP`的源码中只有一次使用，不算是高频，不过在调试时注意到这个了，算是个小技巧吧。可以在调试的时候跳过打上了这个标签的相关代码。具体说明见[官方文档](https://docs.microsoft.com/zh-cn/dotnet/api/system.diagnostics.debuggerstepthroughattribute?redirectedfrom=MSDN&view=netframework-4.8)

> 指示调试器逐句通过代码，而不是进入并单步执行代码。 此类不能被继承。



#### 3. DependsOnAttribute

设置当前模块的依赖模块。这些模块在`ModuleManager`里会依次加载。然后根据依赖的顺序，依次执行每个模块的`PreInitialize`,`Initialize`,`PostInitialize`事件，完成模块的初始化工作。

```c#
[DependsOn(
        typeof(AbpAspNetCoreModule),
        typeof(AbpAspNetCoreDemoCoreModule),
        typeof(AbpEntityFrameworkCoreModule),
        typeof(AbpCastleLog4NetModule),
        typeof(AbpAspNetCoreODataModule)
        )]
```

#### 4. RemoteServiceAttribute

这个也正好是 [【ABP框架笔记】2.ApplicationServices 生成web api](/2020/04/25/abp-services-as-webapi)的一个补充。

[具体的使用](/2020/04/30/abp-controller-to-api/)也写了一个单独的例子

启动过程：

```c#
//1. Abp.Modules.AbpModuleManager 
public virtual void StartModules()
{
    var sortedModules = _modules.GetSortedModuleListByDependency();
    sortedModules.ForEach(module => module.Instance.PreInitialize());
    sortedModules.ForEach(module => module.Instance.Initialize());
    sortedModules.ForEach(module => module.Instance.PostInitialize());
}
//2. Abp.AspNetCore.AbpAspNetCoreModule
/// <summary>
/// 这个方法就是将服务类转成控制器的
/// </summary>
private void AddApplicationParts()
{
    var configuration = IocManager.Resolve<AbpAspNetCoreConfiguration>();
    var partManager = IocManager.Resolve<ApplicationPartManager>();//微软的库，动态的添加视图和控制器
    ...
        //这里就是将服务类转换为控制器的过程
        var controllerAssemblies = configuration.ControllerAssemblySettings.Select(s => s.Assembly).Distinct();
    foreach (var controllerAssembly in controllerAssemblies)
    {
        partManager.AddApplicationPartsIfNotAddedBefore(controllerAssembly);
    }
    ...
}
//3. Abp.AspNetCore.Mvc.Conventions.AbpAppServiceConvention 核心类
/// <summary>
/// 核心实现
/// </summary>
/// <param name="application"></param>
public void Apply(ApplicationModel application)
{
   foreach (var controller in application.Controllers)
    {
        var tempName = controller.ControllerName;
        var type = controller.ControllerType.AsType();
        var configuration = GetControllerSettingOrNull(type);

        //判断当前控制器是否由服务直接生成，是的话移除后缀。并生成相应的路径配置
        if (typeof(IApplicationService).GetTypeInfo().IsAssignableFrom(type))
        {
            //移除类名的后缀："AppService", "ApplicationService" 
            controller.ControllerName = controller.ControllerName.RemovePostFix(ApplicationService.CommonPostfixes);
            Debug.WriteLine($"controller.ControllerName={controller.ControllerName},configuration.ModuleName:{configuration?.ModuleName}");
            configuration?.ControllerModelConfigurer(controller);

            //把service生成的api模块配置成area，本身的控制器视图页已经有了area的路由配置，无需处理
            ConfigureArea(controller, configuration);
            //配置web api，这个方法内部也使用到了RemoteServiceAttribute
            ConfigureRemoteService(controller, configuration);
        }
        else
        {
            //如果是由控制器类实例得来的，判断是否有RemoteService标签。如果有，也要生成相应的 路径。
            var remoteServiceAtt = ReflectionHelper.GetSingleAttributeOrDefault<RemoteServiceAttribute>(type.GetTypeInfo());
            if (remoteServiceAtt != null && remoteServiceAtt.IsEnabledFor(type))
            {
                ConfigureRemoteService(controller, configuration);
            }
        }
    }
}
```

