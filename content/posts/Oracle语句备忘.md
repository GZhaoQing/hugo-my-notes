---
title: Oracle语句备忘
date: 2021-08-18 10:48:44
tags: [Oracle]
categories: [数据库]
---



Oracle详细文档推荐官网 https://docs.oracle.com/

### DBA必备

#### 表空间

```
create tablespace
```



表空间重要视图：

dba_data_files

dba_free_space

dba_tablespaces



查询表空间剩余：

```sql
select "表空间名称",substr((s2)/s1*100,1,6) as "使用率%"  from (
    select "表空间名称",sum(MAX大小GB) s1, sum("已使用GB") s2 from(
        select 
            b.file_name "物理文件名",
            b.tablespace_name "表空间名称",
            b.bytes/1024/1024/1024 "文件大小GB",
            decode(b.maxbytes,0,b.bytes,b.maxbytes)/1024/1024/1024 "MAX大小GB",
            (b.bytes-sum(nvl(a.bytes,0)))/1024/1024/1024 "已使用GB"
        from dba_free_space a, dba_data_files b
        where a.file_id=b.file_id
        group by b.tablespace_name, b.file_name, b.bytes, b.maxbytes
        order by b.tablespace_name
	)WHERE "表空间名称" IN (<填写表空间名称>)  
group by "表空间名称")
    
```



//TODO session  死锁

### 常用SQL

数据合并

merge into <target_table> t1

using ( <source_table> )  t2

on (<condition_of_t1_and_t2>)

when matched then update set <set_sqls>

when not matched then insert ( ... ) values (...)



#### 分区

```SQL
--以时间分区为例
create table <one_table>
(
 ...
 col_date date
)
partition by range(col_date) interval (numtoyminterval(1,'month'))
(
	partition before_part values less than 
    (to_date('2021-01-01 00:00:00','SYYYY-MM-DD HH24:MI:SS'))
)
```

按年  INTERVAL(NUMTOYMINTERVAL(1,'YEAR'))

按天  INTERVAL(NUMTODSINTERVAL(1,'DAY'))



##### 子分区

```
--在range分区基础上再hash分区
partition by range ....  --分区定义
SUBPARTITION BY HASH(last_name)  --hash子分区定义
     SUBPARTITION TEMPLATE       --子分区模板
         (SUBPARTITION a --TABLESPACE ts1, 子分区可以分到不同表空间
          SUBPARTITION b,
          SUBPARTITION c,
          SUBPARTITION d
         )
(
	partition  ...   --分区定义
)

--也可以直接定义hash分区数量
SUBPARTITION BY HASH(last_name) 4
```



##### 分区合并

注意: 范围分区,只能将范围较低的分区,合并到较高的分区

```sql
--官网案例
ALTER TABLE four_seasons 
  MERGE PARTITIONS quarter_one, quarter_two INTO PARTITION quarter_two
  UPDATE INDEXES 
  ONLINE;
```

##### 分区交换

tip: exchange是双向操作! 

在下面例子中, 如果p3无数据, stock_table_3有数据, 数据会转移到p3分区.

如果p3有数据, stock_table_3无数据, 数据会转移到stock_table_3.

如果p3和stock_table_3都有数据, 两者数据也会交换!

exchange要求分区提前建立, 没有创建分区或销毁分区的功能, 只转移数据.

```sql
ALTER TABLE stocks
    EXCHANGE PARTITION p3 WITH TABLE stock_table_3;
```

**警告**: interval分区的表不要手动合并分区, 在exchange操作时会出现bug损坏数据!



#### 在线重定义

要求必须有主键

```
EXEC dbms_redefinition.start_redef_table('eqnt', 'print_media', 'print_media2');
 
DECLARE
 error_count pls_integer := 0;
BEGIN
  dbms_redefinition.copy_table_dependents('eqnt', 'print_media', 'print_media2',
                                          0, true, false, true, false,
                                          error_count);
 
  dbms_output.put_line('errors := ' || to_char(error_count));
END;
/
 
EXEC  dbms_redefinition.finish_redef_table('eqnt', 'print_media', 'print_media2');
```



### 存储过程

//TODO 游标 循环





### 定时任务

oracle有两代定时任务：较早的DBMS_JOB和较新的Schedule系统。

定时相关的实际语句都比较复杂，建议在PL/SQL Developer中用工具创建。

定时任务基本思路，是编写存储过程，确定执行的间隔周期，让存储过程按周期自动重复执行

提示：

​    DBMS_JOB可使用trunc函数定义间隔，例：每日1点：trunc(sysdate+1)+1/24

​    Schedule使用特殊的interval参数定义间隔，例：每日0点：Freq=Daily;ByHour=0
