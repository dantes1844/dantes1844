---
title: 【ABP框架笔记】1.下载示例代码，生成数据库表
date: 2020-04-24 22:30:43
tags: [ABP,.net core 3.0,ef core 3.0]
categories: 
	- [框架]
	- [Web]
---

###  一、官网下载示例代码，过程略

###  二、安装MySql EF 支持包，生成数据库表

#### 1. 安装EF Core的MySQL支持包

在`Laobai.EntityFrameworkCore`项目的依赖项上右键，打开`nuget`管理工具，搜索 `Pomelo.EntityFrameworkCore.MySql`安装即可。

#### 2. 修改代码的连接串，修改配置代码，并生成数据库表

##### 2.1 修改`appsettings.json`的数据库连接字符串。

删除`appsettings.json`中连接串中的`Trusted_Connection=True;`否则会报错，如下内容：

   > Option 'trusted_connection' not supported.

##### 2.2 修改`LaobaiDbContextConfigurer`里的两个方法

将原来的使用`SQLServer`的配置改成使用`MySQL`配置。

   ```c#
public static void Configure(DbContextOptionsBuilder<LaobaiDbContext> builder, string connectionString)
{
    builder.UseMySql(connectionString);
    //builder.UseSqlServer(connectionString);
}
   
public static void Configure(DbContextOptionsBuilder<LaobaiDbContext> builder, DbConnection connection)
{
    builder.UseMySql(connection);
    //builder.UseSqlServer(connection);
}
   ```

##### 2.3 升级数据库

程序包管理控制台里，切换到`Laobai.EntityFrameworkCore`项目文件夹里，执行 `dotnet ef database update`准备生成相应的数据库表结构。这时报了一个错误，如下：

> The EF Core tools version '3.0.0' is older than that of the runtime '3.1.1'. Update the tools for the latest features and bug fixes.

   执行 `dotnet ef -v`得到如下内容：

   > Entity Framework Core .NET Command-line Tools
   > 3.0.0

执行`dotnet tool list -g`得到如下结果，这里如果没有后面的 `-g`全局标记，很可能是找不到这个工具的代码的。

	包 ID     版本  命令
	----
	dotnet-ef 3.0.0 dotnet-ef |

执行升级命令`dotnet tool update dotnet-ef -g`，依然是全局升级，命令行的参数`dotnet-ef`就是上面查到的`ef core`的`ID`

再次执行update数据的命令，依然报错，大概率是`datetime2`类型错误，这是因为默认的`migration`代码是`sqlserver`的，把旧的迁移文件移除，执行`dotnet ef migrations add initialmysqldatabase`生成新的迁移脚本，再执行命令，数据表生成成功。

#### 3. 将启动项目设置为`Host`项目，运行 `VS`,就能顺利打开`swagger`的界面了。