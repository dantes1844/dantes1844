---
title: 数据库表关联更新脚本
date: 2019-09-20 09:36:09
tags: [sqlite,sql,SQL Server]
categories: 
	- [数据库]
---

Sqlite3数据库表关联更新脚本:

```sqlite
UPDATE ComponentType
SET categoryid = (
	SELECT
		id
	FROM
		componentcategory
	WHERE
		ComponentType.CATEGORY = componentcategory.category
)

-- 更新多个字段时，set后面接多个赋值语句即可。
```



SQL Server的脚本

```sql
update table1 
set num1 = t2.num2
FROM table1 t1 INNER JOIN table2 t2 
ON t1.id=t2.pid;  
```