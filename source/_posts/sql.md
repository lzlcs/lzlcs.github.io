---
title: SQL 小结
date: 2023-12-05 13:42:52
mathjax: true
categories:
- Tools
tags:
- SQL
- DataBase
---

# 概述

**数据模型**
1. 层次模型(树)
2. 网状模型(图)
3. 关系模型(矩阵)

**数据类型**

![image](https://github.com/lzlcs/image-hosting/raw/master/image.30bdz680zp60.png)

**安装与启动**

```bash
apt-get install mysql-server
mysql -u root -p
```

# 关系模型

关系模型是一系列二维的表, 表的行是记录, 表的列是字段

字段定义数据类型, 是否允许为 NULL (正常需要避免NULL)

表之间的关系是通过主键和外键维护的

**主键**

同一个表中, 主键不可重复, 所以不可用业务相关的字段作为主键

一般的主键有以下两种
1. 自增整数类型 (INT 和 BIGINT)
2. GUID 算法生成的唯一字符串

> 联合主键: 几个字段共同作为主键 (不推荐)

**外键**

1. 一对多: "多" 的那个表中一个列关联另一个表
2. 多对多: 通过另外的一个表把两个表关联起来(两个一对多)
3. 一对一: 一个列关联到另一个表(拆开一些不常读取的列)

设置外键约束: 
```sql
ALTER TABLE students
ADD CONSTRAINT fk_class_id
FOREIGN KEY (class_id)
REFERENCES classes (id);
```
删除外键约束(不会删除对应列)
```sql
ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```

> 一般不用外键约束降低性能, 让应用程序自己实现外键的功能

**索引**

对某一列或多个列进行预排序从而查询的时候可以直接定位

主键自动生成索引, 可以自定义索引

```sql
ALTER TABLE students
ADD INDEX idx_score (score, name);
```

生成索引的列散列程度越高, 索引的效率越高

索引越多, 查询越快, 修改越慢

唯一索引
```sql
ALTER TABLE students
ADD UNIQUE INDEX uni_name (name);
```
唯一约束(不生成索引)
```sql
ALTER TABLE students
ADD CONSTRAINT uni_name UNIQUE (name);
```

# 查

```sql
SELECT * FROM <表名> WHERE <条件表达式>
SELECT <列1>, <列2> FROM <表名> WHERE <条件表达式>
```

![image](https://github.com/lzlcs/image-hosting/raw/master/image.7e3kvbonypo0.webp)


> `SELECT 100+200;` 可以用于计算, 不过不是强项, 可以用于检查连接状况




