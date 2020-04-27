---
title: 【ABP框架笔记】3.1 本地化模块之资源加载过程
date: 2020-04-26 23:39:31
tags: [ABP,.net core 3.0,ef core 3.0]
categories: 
	- [框架]
	- [Web]
---

[官方文档](https://aspnetboilerplate.com/Pages/Documents/Localization)


##### 1. 模块的加载过程

###### 1)  StartUp类

```C#
	
public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILoggerFactory loggerFactory)
{
    app.UseAbp(); //Initializes ABP framework. Should be called first.
    ...
}

```

###### 2) Abp.AspNetCore.AbpApplicationBuilderExtensions类

```C#
public static void UseAbp([NotNull] this IApplicationBuilder app, Action<AbpApplicationBuilderOptions> optionsAction)
{
    ...
    InitializeAbp(app);//执行各个模块的前置、后置动作的初始化
    ...
}

private static void InitializeAbp(IApplicationBuilder app)
{
    var abpBootstrapper = app.ApplicationServices.GetRequiredService<AbpBootstrapper>();
    abpBootstrapper.Initialize();//执行启动器的初始化方法
    ...
}
```

###### 3)  Abp.AbpBootstrapper类

```
public virtual void Initialize()
{
	...
    _moduleManager.StartModules();//启动模块
    ...
}
```

###### 4) Abp.Modules.AbpModuleManager类

```c#
public virtual void StartModules()
{
    var sortedModules = _modules.GetSortedModuleListByDependency();
    ...
    //加载本地化文件的动作是在核心模块的后置动作(PostInitialize)中
    sortedModules.ForEach(module => module.Instance.PostInitialize());
}
```

###### 5) Abp.AbpKernelModule类

```c#
public override void PostInitialize()
{
    ...
        //解析本地化管理器的实例，并执行它的初始化方法
    IocManager.Resolve<LocalizationManager>().Initialize();
    ...
}
```

###### 6) Abp.Localization.LocalizationManager类

```c#
private void InitializeSources()
{
        ...
        //这里的_configuration.Sources是每个模块（AbpModule的子类）中 PreInitialize（）方法中设定的
        foreach (var source in _configuration.Sources)
        {
            ...

                _sources[source.Name] = source;
            //这里就是按照程序集的名称，读取不同配置文件中的键值对，放入本地化资源中。
            //不同的配置在下面几个章节中分别解读。
            source.Initialize(_configuration, _iocResolver);

            //Extending dictionaries
            if (source is IDictionaryBasedLocalizationSource)
            {
                var dictionaryBasedSource = source as IDictionaryBasedLocalizationSource;
                var extensions = _configuration.Sources.Extensions.Where(e => e.SourceName == source.Name).ToList();
                foreach (var extension in extensions)
                {
                    extension.DictionaryProvider.Initialize(source.Name);
                    foreach (var extensionDictionary in extension.DictionaryProvider.Dictionaries.Values)
                    {
                        dictionaryBasedSource.Extend(extensionDictionary);
                    }
                }
            }
            ...
        }
}
```

##### 2. 本地化资源的种类

- NullLocalizationSource：空的资源解析方式，一种设计方式，避免出现空指针
- ResourceFileLocalizationSource：使用资源文件作为本地化资源文件。
- DictionaryBasedLocalizationSource：使用字典形式的资源文件，也就是将资源文件读成键值对形式的字典，供前后端使用。默认的`abp`示例代码就使用的这种方式。
- MultiTenantLocalizationSource：继承自`DictionaryBasedLocalizationSource`，应该是说多租户模式下，也是使用字典模式的资源文件。

也可以自定义资源的存储方式，然后实现`ILocalizationSource`接口来解析对应的资源文件。

*个人觉得：`xml`模式足够好了，维护很方便，没有必要使用`ResourceFileLocalizationSource`或者自定义模式。*

##### 3. 配置文件的解析方式

目前总共是四种解析方式：

- JsonFileLocalizationDictionaryProvider：从`json`文件中读取字典
- JsonEmbeddedFileLocalizationDictionaryProvider：从嵌入的`json`文件中读取字典，这个时候要将文件属性设置成"嵌入的资源"
- XmlFileLocalizationDictionaryProvider：从`xml`文件中读取字典
- XmlEmbeddedFileLocalizationDictionaryProvider：从嵌入的`xml`文件中读取字典，这个时候要将文件属性设置成"嵌入的资源"



##### 4. 如何在系统中配置要使用的资源

在添加配置项的时候，应当设置相关解析的属性。以`Abp.AbpKernelModule`类为例，它配置了嵌入的`xml`资源文件及解析方式：

```c#
 
 private void AddLocalizationSources()
 {
 	//要读取本地化资源的程序集
    var assembly = typeof(AbpKernelModule).GetAssembly();
    //资源文件除文件名外的完整命名空间
    var rootNameSpaces = "Abp.Localization.Sources.AbpXmlSource";
    //资源解析器
    var provider = new XmlEmbeddedFileLocalizationDictionaryProvider(assembly, rootNameSpaces);
    //资源
    var source = new DictionaryBasedLocalizationSource(AbpConsts.LocalizationSourceName, provider);
    Configuration.Localization.Sources.Add(source);
}
```

使用资源文件时要注意，文件名以`-`分割，而不要使用`.`，因为`.`会造成错误的命名空间导致找不到资源文件。例如：使用`bussiness-cn.xml`或者`bussiness-cn.json`，而非~~`bussiness.cn.xml`~~等。

另外，还可以根据模块(`module`)设置不同的资源，例如：在业务类可以设置一个资源文件，在`web`层设置另外的资源，这样可以将资源分开管理。最终都是在同一个地方加载到配置里的。

`xml`或者`json`文件都必须是`Unicode`格式，因为最终`abp`是使用`Unicode`编码读取的文件流，然后解析内容。

```c#
public static string ReadStringFromStream(Stream stream)
{
    var bytes = stream.GetAllBytes();
    var skipCount = HasBom(bytes) ? 3 : 0;
    return Encoding.UTF8.GetString(bytes, skipCount, bytes.Length - skipCount);
}
```

##### 5. 其他

###### 5.1 abp的模块(module)核心：

从以上过程也能看出来，`abp`在模块的核心内容是四个执行过程，分别是

- PreInitialize：执行一些在系统初始化前需要配置的内容，例如本地化资源解析方式。

- Initialize：执行一些系统初始化中执行的内容，例如：为每个程序集设置注入内容。

- PostInitialize：执行一些系统初始化完成后执行的内容，例如加载本地化资源的内容。

- Shutdown：在系统崩溃时执行的一些内容，例如在系统崩溃时，有些资源需要手动释放。


