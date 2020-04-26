---
title: Visual Studio 2019 安装ABP项目
date: 2019-11-12 08:41:30
tags: [ABP,ef core 3.0,.net core 3.0]
categories: 
	- [工具]	
	- [框架]
---



1. Pomelo.EntityFrameworkCore.MySql不是.net core 3.0会报如下错:

> Method 'get_Info' in type 'Pomelo.EntityFrameworkCore.MySql.Infrastructure.Internal.MySqlOptionsExtension' from assembly 'Pomelo.EntityFrameworkCore.MySql, Version=2.2.0.0, Culture=neutral, PublicKeyToken=null' does not have an implementation.

只能去安装对应的3.0包了，然而在今天(2019年11月12日)的时候，Nuget管理工具里最高的发布版本是2.2.6,那就装个RC版的吧。要注意的是，它的官方文档上说了，这个命令只支持VS2017+并且Nuget是V4.3以上的版本才行。命令如下：

```c#
Install-Package Pomelo.EntityFrameworkCore.MySql -Version 3.0.0-rc2.final
```

2. .net core 3.0将ef工具抽出去了。

MySQL包装完之后，执行命令，又崩了。错误如下：



> dotnet ef 
> dotnet : 无法执行，因为找不到指定的命令或文件。
> 所在位置 行:1 字符: 1

[官网](https://docs.microsoft.com/en-us/ef/core/what-is-new/ef-core-3.0/breaking-changes#dotnet-ef)是这么解释的，也提供了相应的解决办法，那么就执行全局工具安装脚本吧。问题解决。



> **New behavior**
>
> Starting in 3.0, the .NET SDK does not include the `dotnet ef` tool, so before you can use it you have to explicitly install it as a local or global tool.
>
> **Why**
>
> This change allows us to distribute and update `dotnet ef` as a regular .NET CLI tool on NuGet, consistent with the fact that the EF Core 3.0 is also always distributed as a NuGet package.
>
> **Mitigations**
>
> To be able to manage migrations or scaffold a `DbContext`, install `dotnet-ef` as a global tool:
>
> consoleCopy
>
> ```console
>$ dotnet tool install --global dotnet-ef
> ```
>
> You can also obtain it a local tool when you restore the dependencies of a project that declares it as a tooling dependency using a [tool manifest file](https://github.com/dotnet/cli/issues/10288).

3. MySQL本身的错误。

   哗啦啦一堆脚本执行，没问题，最后咔，出错了。

   

> MySql.Data.MySqlClient.MySqlException (0x80004005): Specified key was too long; max key length is 767 bytes
>  ---> MySql.Data.MySqlClient.MySqlException (0x80004005): Specified key was too long; max key length is 767 bytes
>    at MySqlConnector.Core.ResultSet.ReadResultSetHeaderAsync(IOBehavior ioBehavior) in C:\projects\mysqlconnector\src\MySqlConnector\Core\ResultSet.cs:line 49
   >    at MySql.Data.MySqlClient.MySqlDataReader.ActivateResultSet() in C:\projects\mysqlconnector\src\MySqlConnector\MySql.Data.MySqlClient\MySqlDataReader.cs:line 130
   >    at MySql.Data.MySqlClient.MySqlDataReader.CreateAsync(CommandListPosition commandListPosition, ICommandPayloadCreator payloadCreator, IDictionary`2 cachedProcedures, IMySqlCommand command, CommandBehavior behavior, IOBehavior ioBehavior, CancellationToken cancellationToken) in C:\projects\mysqlconnector\src\MySqlConnector\MySql.Data.MySqlClient\MySqlDataReader.cs:line 391
   >    at MySqlConnector.Core.CommandExecutor.ExecuteReaderAsync(IReadOnlyList`1 commands, ICommandPayloadCreator payloadCreator, CommandBehavior behavior, IOBehavior ioBehavior, CancellationToken cancellationToken) in C:\projects\mysqlconnector\src\MySqlConnector\Core\CommandExecutor.cs:line 62