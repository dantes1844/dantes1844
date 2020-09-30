---
title: ABP 服务类使用基本数据类型的参数问题
permalink: ABP_service_use_primitive_type_parameters
date: 2020-09-28 12:36:10
tags: [ABP, Web API]
---

今天在写一个业务类时，往里传入了一个整型的参数，然而`Service`获得的参数始终是0。



查看了相关文档之后，发现，`web api`本身其实是支持基本数据类型的，而`abp`内部则没有对简单类型进行处理。所以无法获取参数，必须将其封装成一个`dto`类以获取值。

这样的操作似乎有点浪费，为了一个简单的参数封装一个`dto`.

web api 参数绑定方式：

> ### When do we use which?
>
> Here are the basic rules to determine whether a parameter is read with model binding or a formatter:
>
> 1. If the parameter has no attribute on it, then the decision is made purely on the parameter’s .NET type. **“Simple types” uses model binding.** Complex types uses the formatters. A “simple type” includes: [primitives](https://msdn.microsoft.com/en-us/library/system.type.isprimitive.aspx), TimeSpan, DateTime, Guid, Decimal, String, or something with a TypeConverter that converts from strings.
> 2. You can use a [FromBody] attribute to specify that a parameter should be from the body.
> 3. You can use a [ModelBinder] attribute on the parameter or the parameter’s type to specify that a parameter should be model bound. This attribute also lets you configure the model binder. [FromUri] is a derived instance of [ModelBinder] that specifically configures a model binder to only look in the URI.
> 4. The body can only be read once. So if you have 2 complex types in the signature, at least one of them must have a [ModelBinder] attribute on it.

相关连接:

[abp 不支持基本类型数据参数的解释](https://github.com/aspnetboilerplate/aspnetboilerplate/issues/551)

[web api 支持的参数类型](https://docs.microsoft.com/en-us/archive/blogs/jmstall/how-webapi-does-parameter-binding)

