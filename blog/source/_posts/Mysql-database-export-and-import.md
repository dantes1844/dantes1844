---
title: MySQL的导出导入笔记
date: 2019-09-03 20:44:13
tags: [MySql]
categories: 
	- [数据库]
---

计划将阿里云上的MySQL数据库导一份到本地。记录下踩坑过程。

### 1. 导出

执行以下脚本，输入密码，即可将数据库导出到指定的目录下。

```sql
mysqldump -u root -p dbname > mydb.sql
```

### 2. 创建本地数据库

```sql
create database mytest;
```

执行上面的脚本，创建数据库。执行时可能会出现以下错误

>  `Access denied ``for` `user` `'root'``@``'%'` `to` `database` `'mytest'`

原因：**创建完数据库后，需要进行授权，在本地访问一般不会存在这个问题。**

授权操作:



```sql
grant all on xxx.* to 'root'@'%' identified by 'password' with grant option
```

其中***表示数据库名称



### 3. 执行导入操作



```sql
mysql -u root -p doubanjiang < D:/codes/database/doubanjiang.sql
```

执行时依然报错，`ERROR at line 1356: Unknown command '\''.`

这个是编码问题造成的，指定脚本的执行编码就可以解决问题，脚本如下:

``````sql
mysql -u root -p  --default-character-set=utf8 doubanjiang < D:/codes/database/doubanjiang.sql
``````

报错：`ERROR 1071 (42000) at line 1501: Specified key was too long; max key length is 767 bytes`

执行如下脚本



```sql
--修改最大索引长度限制
set global innodb_large_prefix=1; 
set global innodb_file_format=BARRACUDA;
-- 添加
set global innodb_file_format_max=BARRACUDA;
```

导入成功。



参考文档:

[1.Mysql Specified key was too long; max key length is 767 bytes](https://www.cnblogs.com/wang-yaz/p/10942869.html)](https://www.cnblogs.com/wang-yaz/p/10942869.html)