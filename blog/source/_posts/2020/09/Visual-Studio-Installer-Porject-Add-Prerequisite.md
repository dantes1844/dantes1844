---
title: Visual Studio Installer Porject 增加指定的Framework包
permalink: Visual_Studio_Installer_Porject_Add_Prerequisite
date: 2020-09-09 16:28:14
tags: [Visual Studio]
---



在制作一个安装包之后，本地运行没问题，客户始终无法运行。最终发现本地是用的`.net Framework4.8`，客户是`4.6.2`，编译器生成的文件里有一部分不在根目录。或许是不同版本的`Framework`编译的结果有差异吧。这个待查。

于是准备将`Framework`包打进安装包里，在安装项目上右键，选择属性栏

![属性](属性.png)

在弹出的窗体上点击`Prerequisites`按钮，在弹出的界面里勾选相关的版本及获取方式。这里选择第二个，离线包模式。

![选择获取方式](选择获取方式.png)

接下来就是关键操作。根据[官方文档](https://docs.microsoft.com/en-us/visualstudio/deployment/how-to-include-prerequisites-with-a-clickonce-application?view=vs-2017&redirectedfrom=MSDN)。主要几个步骤：

1. 找到`%ProgramFiles(x86)%\Microsoft SDKs\ClickOnce Bootstrapper\Packages\`路径
2. 找到对应版本的文件夹，例如这里就是`DotNetFX48`
3. 在里面打开配置文件，找到包的下载路径，下载相关文件，保存到本地文件夹。
4. 最关键一步：将下载的文件复制到`DotNetFX48`文件夹里放着，***不要***放在任何语言包内

![下载路径](下载路径.png)

完成以上步骤后，再生成打包项目时，应该就能将需要的`Framework`文件打包到生成的安装文件夹目录里，一同拷贝发送用户，即可自动安装需要的`Framework`。

