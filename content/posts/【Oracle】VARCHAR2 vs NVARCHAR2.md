---
title: "【Oracle】VARCHAR2 vs NVARCHAR2"
date: 2021-10-25T10:55:37+08:00
draft: true
tags: [数据类型]
categories: [Oracle, DataBase]
---



初接触数据库表设计的同学，可能都在VARCHAR2和NVARCHAR2之间纠结过。

从一个Java开发角度看，其实只要做好数据库字符集的统一，无脑选VARCHAR2没有什么问题。

共同点：

根据Oracle文档的介绍，VARCHAR2和NVARCHAR2上限都是4000bytes。

重要区别：

##### VARCHAR2 

VARCHAR2有2种定义方式，VARCHAR2(10 BYTE)和VARCHAR2(10 CHAR)，

默认VARCHAR2(2000)用的是BYTE！

Oracle用AL32UTF8字符集，中文是3个byte，VARCHAR2(10)只能存储3个汉字，还浪费1byte；VARCHAR2(10 CHAR)能存储10个汉字。 但要记住VARCHAR2有4000bytes上限，所以VARCHAR2(2000 CHAR)也不能存2000汉字(最多1333)。

##### NVARCHAR2

NVARCHAR2则定义的是字符数，只存储Unicode字符，能存2000汉字。



VARCHAR2使用数据库字符集，NVARCHAR2使用国家字符集

如下，在Oracle配置里是不同的两项

```
NLS_CHARACTERSET = AL32UTF8
NLS_NCHAR_CHARACTERSET = UTF8
```





一些坑：

环境：SpringBoot2.5.2，Oracle 11g，hibernate5.4.32

现象：实际开发中用JPA间接使用hibernate，在构造NVARCHAR2字段的模糊查询时，存在ORA-01425的报错，从结果来看，可能是ESCAPE子句默认的"\\"字符被自动转义成了"\\\\\"，具体原因有待进一步跟踪。

关键信息如下：

```
v_cata_ser0_.name like ? escape ?

skipping....

TRACE org.hibernate.type.descriptor.sql.BasicBinder - binding parameter [1] as [VARCHAR] - [%房产%]
TRACE org.hibernate.type.descriptor.sql.BasicBinder - binding parameter [2] as [CHAR] - [\]
WARN  org.hibernate.engine.jdbc.spi.SqlExceptionHelper - SQL Error: 1425, SQLState: 22019
ERROR org.hibernate.engine.jdbc.spi.SqlExceptionHelper - ORA-01425: 转义符必须是长度为 1 的字符串
```

而在VARCHAR2上则没有这种现象。

最快处理：把NVARCHAR2字段to_char成VARCHAR2。(在UNION合并表时也会有这样的技巧)





