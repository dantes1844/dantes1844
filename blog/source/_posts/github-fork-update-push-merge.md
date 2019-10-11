---
title: Github上fork代码并修改，之后提交到自己的版本库流程
date: 2019-10-10 14:13:47
tags: [Github]
---

1. fork 代码，在Github上操作即可。



2. clone 代码



3. 根据相关的标签或者分支，切换自己的分支出来。这一步尤为关键，之前我就是切换到一个提交的版本上，然后修改代码，再push时就始终无法提交成功。

   当以tag标签切换分支之后，在分支上修改的代码就能正确提交了。

   `git branch <branchname> <tagname>`

   `git checkout <branchname>`

   这里也能看出来，Github里"万物皆分支"。

   > 参考1

4. 更新自己的代码

   

5. 提交代码到Github。在分支内，执行`git push origin <branchname>`这样在Github上fork的代码里看到提交的的记录了。但这个时候首页的绿色小点应该是看不到的，需要手动合并一下分支，并且删除相关分支。或者采用下面的步骤，先合并之后，再提交。

6. 合并分支到主干。

> 参考2
>
> 于复杂的系统，我们可能要开好几个分支来开发，那么怎样使用git合并分支呢？
>
> 合并步骤：
>
> 1、进入要合并的分支（如开发分支合并到master，则进入master目录）
>
> git pull
>
> 2、查看所有分支是否都pull下来了
>
> git branch -a
>
> 3、使用merge合并开发分支
>
> git merge 分支名
>
> 4、查看合并之后的状态
>
> git status 
>
> 5、有冲突的话，通过IDE解决冲突；
>
> 6、解决冲突之后，将冲突文件提交暂存区
>
> git add 冲突文件
>
> 7、提交merge之后的结果
>
> git commit 
>
> 如果不是使用git commit -m "备注" ，那么git会自动将合并的结果作为备注，提交本地仓库；
>
> 8、本地仓库代码提交远程仓库
>
> git push



Reference:

[1. 从已有tag上创建分支](https://www.jianshu.com/p/ef23d15742a9)

[2. 合并分支到主干](https://blog.csdn.net/BlueBirdssh/article/details/88393751)