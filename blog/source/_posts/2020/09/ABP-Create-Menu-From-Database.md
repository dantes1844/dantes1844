---
title: ABP 从数据库中创建菜单
permalink: ABP_Create_Menu_From_Database
date: 2020-09-11 13:57:26
tags:
---

[ABP 支持组给出的建议](https://support.abp.io/QA/Questions/224/AngularHow-to-build-all-menus-from-the-database)

> The current design of the menu does not consider persistence to the database, you need to do some work for this.
>
> 1.  You need to create an entity that is compatible with `ApplicationMenu`.
> 2.  Create a service that persists all menus to the database when the application is initialized(If the menu is not in the database table)
> 3.  Custom `DatabaseMenuManager` to replace `MenuManager` and get the menu from the database(should be filtered based on the current user).





### 实践

#### 1. 创建菜单服务类

![接口、类及dto文件](services.png)

在`Application`项目里增加如下结构。主要的类文件`MenuService`是一个普通的业务类，继承自`WindElectricityAppServiceBase`以及自定义的接口`IMenuService`。接口继承自`IAsyncCrudAppService<MenuDto>`，这样就包含了默认要实现的几个`CRUD`方法。其中获取菜单的方法核心代码如下。

```c#
public async Task<ListResultDto<MenuDto>> GetMenuForProviderAsync()
        {
            var menus = await _menuRepository.GetAllListAsync(c => !c.IsDeleted);
            
            //return await Task.FromResult(new ListResultDto<MenuDto>(ObjectMapper.Map<List<MenuDto>>(menus)));
            var result = new List<MenuDto>();
            foreach (var menu in menus)
            {
                result.Add(new MenuDto()
                {
                    Icon = menu.Icon,
                    Url = menu.Url,
                    DisplayName = menu.DisplayName,
                    Name = menu.Name,
                    Id = menu.Id,
                    ParentMenu = menu.ParentMenu,
                    ParentMenuId = menu.ParentMenuId,
                    RequiresAuthentication = menu.RequiresAuthentication,
                    SubMenus = menu.SubMenus
                });
            }
            return new ListResultDto<MenuDto>(result);
        }
```

**注意事项，在这个时候，是不能调用`ObjectMapper`的，原因是：`AutoMapper`是在`AbpAutoMapperModule`的`PreInitialize`中替换成有效实现类（`AutoMapperObjectMapper`），而`AutoMapperObjectMapper`又依赖于`IMapper`的注入，该接口的注入是在`AbpAutoMapperModule`的`PostInitialize`方法中，  因`AbpAutoMapperModule`依赖于`AbpKernalModule`，该方法在核心模块的`PostInitialize`之后执行，所以在核心模块的`PostInitialize`中，  获取菜单时，要使用`ObjectMapper`会报错。**

##### 1.1 Castle.Windsor的属性注入

[参考官方文档](https://github.com/castleproject/Windsor/blob/master/docs/how-properties-are-injected.md)

核心内容：

- Has 'public' accessible setter：必须是public的
- Is an instance property：属性而非字段
- If `ComponentModel.InspectionBehavior` is set to `PropertiesInspectionBehavior.DeclaredOnly`, is not inherited：
- Does not have parameters：没有参数
- Is not annotated with the `Castle.Core.DoNotWireAttribute` attribute：没有打标签`Castle.Core.DoNotWireAttribute`

##### 1.2 abp自定义的PropertiesDependenciesModelInspector

`abp`自定义了一个`AbpPropertiesDependenciesModelInspector`继承自`PropertiesDependenciesModelInspector`，该类仅仅是增加一个判断，以`Microsoft`开头的属性不进行注入。在`abp`的代码库中，`AbpAspNetCoreDemo`的`Startup`中的`ConfigureServices`方法中，也演示了如何调用自定义的`PropertiesDependenciesModelInspector`。当然我们也可以扩展自己的类，按照自己的需要决定属性如何注入。

```
var propInjector = options.IocManager.IocContainer.Kernel.ComponentModelBuilder
                    .Contributors
                    .OfType<PropertiesDependenciesModelInspector>()
                    .Single();

                options.IocManager.IocContainer.Kernel.ComponentModelBuilder.RemoveContributor(propInjector);
                options.IocManager.IocContainer.Kernel.ComponentModelBuilder.AddContributor(new AbpPropertiesDependenciesModelInspector(new DefaultConversionManager()));
           
```



#### 2. 增加自定义菜单Provider

在`web`项目的`StartUp`文件夹中增加一个自定义的`NavigationProvider`实现类。核心代码如下：

```c#
public class CustomDbNavigationProvider : NavigationProvider
    {
        public override async void SetNavigation(INavigationProviderContext context)
        {
            var menuService = IocManager.Instance.Resolve<IMenuService>();
            var menus = await menuService.GetMenuForProviderAsync();
            foreach (var menu in menus.Items)
            {
                if (menu.ParentMenuId.HasValue)
                {
                    continue;
                }
                var menuOfProvider = new MenuItemDefinition(
                    menu.Name,
                    L(menu.DisplayName),
                    url: menu.Url,
                    icon: menu.Url,
                    requiresAuthentication: menu.RequiresAuthentication
                );
                if (!menu.SubMenus.IsNullOrEmpty())
                {
                    foreach (var menuSubMenu in menu.SubMenus)
                    {
                        var submenuOfProvider = new MenuItemDefinition(
                            menuSubMenu.Name,
                            L(menuSubMenu.DisplayName),
                            url: menuSubMenu.Url,
                            icon: menuSubMenu.Url,
                            requiresAuthentication: menuSubMenu.RequiresAuthentication
                        );
                        menuOfProvider.AddItem(submenuOfProvider);
                    }
                }
                context.Manager.MainMenu.AddItem(menuOfProvider);
            }
        }

        //...
    }
```

#### 3. 修改默认的菜单Provider

修改`Startup`文件夹中模块类的`PreInitialize`方法。将调用默认的`Provider`改成自定义的`Provider`

```c#
public override void PreInitialize()
{
    Configuration.Navigation.Providers.Add<CustomDbNavigationProvider>();
}
```

#### 4. 增加种子数据

如果有需要，在项目的`EntityFrameworkCore`层中增加相应的种子数据类及方法，并在`SeedHelper`中进行调用。这个就是根据表自动进行拼凑实体，调用相关的保存方法即可。代码略。

![文件示例](seed.png)