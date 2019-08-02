---
title: hexo+github搭建博客关键点
date: 2019-08-01 14:00:18
tags: Hexo
---

### 1. 参考以下内容：

1.[简书](https://www.jianshu.com/p/eded1dd2d794)

2.[Hexo官网](https://hexo.io/zh-cn/)

3.[Hexo官网部署文档](https://hexo.io/zh-cn/docs/deployment)

4.[解决Hexo在Github上部署后个人页面404的问题](https://blog.csdn.net/qq32933432/article/details/87955133)

### 2. 主要步骤及相关脚本

1. 安装Hexo脚手架 
	
```bash
npm install hexo-cli -g
```

2. 初始化一个Hexo项目，名称为`blog` 

```bash
hexo init blog
```

3. 切换到项目文件夹里 
	
```bash
cd blog
```

4. 执行安装，***这一步的作用是啥我也不知道，需要研究下官方文档***

```bash
npm install
```

5. 启动博客程序，会打开默认端口4000.在浏览器里输入`http://localhost:4000`即可访问了。

```bash
hexo server
```

6. 执行清理旧的文件并发布新的

```bash
hexo clean && hexo deploy
``` 

### 3. 注意事项

1.配置文件(_config.yml)的键值对之间有个空格
	
2.Github的Repository的名字一定是自己的用户名+".github.io"格式,例如我的就是"dantes1844.github.io".

### 4. 版本控制

参考两篇文章

[1.使用hexo，如果换了电脑怎么更新博客？ - CrazyMilk的回答 - 知乎](https://www.zhihu.com/question/21193762/answer/79109280)

[2.hexo博客同步管理及迁移 - 简书](https://www.jianshu.com/p/fceaf373d797)
	
其实第二篇文章也是参考第一个的，不过它的解释更详细，让人明白为什么要这样做。我是看他的文章知道原理，然后按照第一个的步骤实现代码上传的。囧~~

另外，如果使用Hexo主题，从Github上更新的代码，在主题文件夹里，相应的主题包下面的.git文件要删掉，否则该文件夹下面的代码无法正常提交到Github。