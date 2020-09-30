---
title: asp.net core 集成 Unity WebGL 类型匹配问题
permalink: Asp-Net-Core-Unity-WebGL-MIME-type-problem
date: 2020-09-30 14:31:36
tags: [.net core 3.0]
---

项目中需要集成`Unity`打包的`WebGL`内容，按照示例代码，将`js`和文件包引入之后，部署到`IIS`时，会报如下错误
> An error occurred running the Unity content on this page.See your browser Javascipt console for more info .The error was:
Uncaught SyntaxError Unexpected end of input



打开浏览器的控制台就能看到，是`WebGL`包里的后缀为`unityweb`的文件无法加载。主要问题就是对应的`MIME`类型无法解析。

在[unity的官网](https://forum.unity.com/threads/webgl-for-asp-net-core.790070/)也有相关解决方案。核心代码：

```c#
//将以下代码粘贴替换到Startup.cs的Configure方法中

var provider = new FileExtensionContentTypeProvider();
// Add new mappings
provider.Mappings[".unityweb"] = "application/unityweb";

app.UseStaticFiles(new StaticFileOptions()
                   {
                       FileProvider = new PhysicalFileProvider(_env.WebRootPath),
                       ContentTypeProvider = provider
                   });
```



翻看了下`FileExtensionContentTypeProvider`类的源码。内容很简单

1. `Dictionary<string, string> Mappings`属性

2.  一个无参构造函数，其中无参构造会将`IIS`中已经支持的`MIME`映射类型自动加入`Mappings`字典中去。有380个类型的数据映射，但是没有`unityweb`类型。

3.  一个有参构造，可以传入字典类型的数据。

4.  一个获取映射类型的方法`TryGetContentType`。

   

所以上面的解决方案也很清晰，就是加入相关的`MIME`映射。