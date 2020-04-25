---
title: Visual Studio 2017 创建 Install Project笔记
date: 2019-11-08 10:19:13
tags: [Install Project,C#]
categories: 
	- [工具]
---

#### 1.建立Install Project项目过程

VS2013的安装项目是一个exe工具，到了2017里，是一个插件了，需要在VS-扩展和更新里安装，或者下载VSIX包按照。

创新项目的过程并不是特别复杂，参考下面两篇文章。

[参考连接1](https://blog.csdn.net/dog123xuheyin/article/details/85008071)   [参考链接2](https://www.cnblogs.com/duanweishi/p/11114332.html)

### 2.建立完项目后不能编译的解决方法:

我的Project插件版本是`0.9.3`

[参考连接](https://stackoverflow.com/questions/44907844/windows-installer-project-in-visual-studio-2017)

```bash
The problem was resolved on an x64 system by running:

regsvr32.exe /u "C:\Program Files (x86)\Common Files\microsoft shared\MSI Tools\mergemod.dll"

regsvr32.exe "C:\Program Files (x86)\Common Files\microsoft shared\MSI Tools\mergemod.dll"
```

### 3.生成了安装包之后，注册表的`InstallLocation`为空的解决办法。

两个办法的思路其实是一致的，增加自定义的安装类，重新默认的安装方法。

[参考连接1,老外的方法](http://www.mikebevers.be/blog/2010/01/setup-project-product-installlocation-in-registry-is-empty/)

[参考连接2（这个参考连接2与安装里的参考连接2是同一个）](https://blog.csdn.net/smallbabylong/article/details/78756530)

最终代码，需要添加`using Microsoft.Win32;`的引用，另外还要添加`System.Configuration.Install`的引用。



```c#
[RunInstaller(true)]
    public class InstallerPatch : Installer
    {
        private const string Win64RegistryPath = @"SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall";
        private const string Win32RegistryPath = @"SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall";

        public override void Install(IDictionary stateSaver)
        {
            base.Install(stateSaver);

            stateSaver.Add("TargetDir", Context.Parameters["DP_TargetDir"]);
            stateSaver.Add("ProductID", Context.Parameters["DP_ProductID"]);
        }

        public override void Commit(IDictionary savedState)
        {
            base.Commit(savedState);

            var productId = savedState["ProductID"].ToString();

            var applicationRegistry = Registry.LocalMachine.OpenSubKey($"{Win64RegistryPath}\\{productId}", true);

            if (applicationRegistry != null)
            {
                applicationRegistry.SetValue("InstallLocation", savedState["TargetDir"].ToString());
                applicationRegistry.Close();
            }

            applicationRegistry = Registry.LocalMachine.OpenSubKey($"{Win32RegistryPath}\\{productId}", true);

            if (applicationRegistry != null)
            {
                applicationRegistry.SetValue("InstallLocation", savedState["TargetDir"].ToString());
                applicationRegistry.Close();
            }
        }
    }
```

参考连接2里的方法始终不能正确执行，老外的链接是可以的。有个注意的地方是，老外重写的两个方法是Commit和Install，所以要在安装包的自定义的动作里分别指定这两个，并且两个的CustomAction都要进行设置。

以上是几个关键点，昨天踩了一上午的坑，终于解决问题。记录下。