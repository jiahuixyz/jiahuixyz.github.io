---
layout: post
title: Oracle常用语法
date: 2019-10-26 01:01:47
tags:
---

# DDL修改表结构

```sql
--添加字段的语法：
alter table tablename add (column datatype [default value][null/not null],….);
--修改字段的语法：
alter table tablename modify (column datatype [default value][null/not null],….);
--删除字段的语法：
alter table tablename drop (column);
```

# 创建序列

```sql
 -- 创建序列
 create sequence Student_stuId_Seq
 increment by 1
 start with 1
 minvalue 1
 maxvalue 999999999;
 
 -- 调用序列
 insert into Student(stuId,Stuname) values(Student_stuId_Seq.Nextval,'张三');
```

原文：[http://www.cnblogs.com/xielong/p/8950999.html](http://www.cnblogs.com/xielong/p/8950999.html)
<!--more-->
# start with connect by prior
Oracle的start with connect by prior是根据条件递归查询"树"，分为四种使用情况： <br>
+ 第一种：`start with 子节点ID='n' connect by prior 子节点ID = 父节点ID`<br>
查询结果为自己（子ID为n）所有的后代节点（包括自己）。<br>
+ 第二种：`start with 子节点ID='n' connect by 子节点ID = prior 父节点ID`<br>
查询结果为自己（子ID为n）所有的前代节点（包括自己）。<br>
+ 第三种：`start with 父节点ID='n' connect by prior 子节点ID = 父节点ID`<br>
查询结果为自己（父ID为n，可能有多个）所有的后代节点（包括自己）。<br>
+ 第四种：`start with 父节点ID='n' connect by 子节点ID = prior 父节点ID`<br>
查询结果为自己（父ID为n，可能有多个）的所有的前代节点（包括自己）。<br>

> 注意子ID不会重复，但父ID会重复！注意包括自己这个词！

>如`select * from t_menu t start with t.menu_id = '1' connect by t.parent_id = prior t.menu_id`这条SQL，
逻辑为：首先找到menu_id为1的记录，接着查找parent_id等于上一条记录（首先找到的记录）的menu_id的记录，接着对找到的记录重复这个过程。

>如果有where 条件，执行顺序为先执行start with connect by prior，然后再按照where条件进行过滤。

```sql
select * from mdm_organization o where 条件 start with o.org_parent_code='10000008' connect by o.org_code = prior o.org_parent_code<br>
```

- prior ： 表示上一层级的标识符。经常用来对下一层级的数据进行限制。不可以接伪列
- level ：伪列，表示当前深度

用法：[https://blog.csdn.net/nayi_224/article/details/79811185](https://blog.csdn.net/nayi_224/article/details/79811185) 

# merge into
>如果符合条件就执行更新，不符合条件则插入

```sql
MERGE INTO [your table-name] [rename your table here] 
USING ( [write your query here] )[rename your query-sql and using just like a table] 
ON ([conditional expression here] AND [...]...) 
WHEN MATCHED THEN [here you can execute some update sql or something else ] 
WHEN NOT MATCHED THEN [execute something else here ! ] 
```

示例：
```sql
MERGE INTO  TEST T1
USING (SELECT '2' as ID, 'newtest2' as NAME FROM dual) T2 on (T1.ID=T2.ID)
WHEN MATCHED THEN UPDATE SET T1.NAME=T2.NAME
WHEN NOT MATCHED THEN  INSERT (T1.ID, T1.NAME) VALUES (T2.ID, T2.NAME ); 
```

>merge into在更新后还可以执行删除

# with as

```sql
--相当于建了个e临时表
with e as (select * from scott.emp e where e.empno=7499)
select * from e;
```

# decode

```sql
--语法：
DECODE(value,if1,then1,if2,then2,if3,then3,...,else)
```

表示如果value等于if1时，DECODE函数的结果返回then1,...,如果不等于任何一个if值，则返回else。

# union all

- UNION去重且排序
- UNION ALL不去重不排序（效率高）

# first_rows 优化器模式

```
 select * from (
     select /*+ first_rows */
     rownum rn,a.object_name
     from page_test a,
          page_test b,
          page_test c
     where a.id=b.id
     and b.id=c.id
     and rownum<=5
   ) where rn>0;
```

# 分页

无ORDER BY排序的写法：

```
 SELECT *
  FROM (SELECT ROWNUM AS rowno, t.*
          FROM emp t
         WHERE hire_date BETWEEN TO_DATE ('20060501', 'yyyymmdd')
                             AND TO_DATE ('20060731', 'yyyymmdd')
           AND ROWNUM <= 20) table_alias
 WHERE table_alias.rowno >= 10;
```

有ORDER BY排序的写法：

```
 SELECT *
  FROM (SELECT tt.*, ROWNUM AS rowno
          FROM (  SELECT t.*
                    FROM emp t
                   WHERE hire_date BETWEEN TO_DATE ('20060501', 'yyyymmdd')
                                      AND TO_DATE ('20060731', 'yyyymmdd')
                ORDER BY create_time DESC, emp_no) tt
         WHERE ROWNUM <= 20) table_alias
 WHERE table_alias.rowno >= 10;
```

# exists & in

1、select * from T1 where 
exists(select 1 from T2 where T1.a=T2.a) ;<br>
T1数据量小而T2数据量非常大时，T1<<T2 时，exists 的查询效率高。（查询T2表count(T1)次）

2、select * from T1 where T1.a in (select T2.a from T2) ;<br>
T1数据量非常大而T2数据量小时，T1>>T2 时，in 的查询效率高。（遍历内存count(T2)次）

>exists需要查询数据库，in在内存中遍历。

# sign(n)

>取数字n的符号,大于0返回1,小于0返回-1,等于0返回0

使用sign函数解决大于小于的问题：

```sql
select decode(sign(colA - colB) , -1 , colA + colB , colA) from table_name
```

更复杂的示例：
```sql
-- 需求：员工的入职日期向前推12个月、20个月，得到一个时间段，若该时间段与所提供的另一个时间段重合，则在排序中将该员工排到前面。
select emp.*,
       add_months(emp.hiredate, -20),
       add_months(emp.hiredate, -12),
       decode(sign(add_months(emp.hiredate, -12) -
                   to_date('19790101', 'yyyyMMdd')) +
              sign(to_date('19791231', 'yyyyMMdd') -
                   add_months(emp.hiredate, -20)),
              2,
              0,
              1) sort
  from emp
 order by decode(sign(add_months(emp.hiredate, -12) -
                      to_date('19790101', 'yyyyMMdd')) +
                 sign(to_date('19791231', 'yyyyMMdd') -
                      add_months(emp.hiredate, -20)),
                 2,
                 0,
                 1)

-- (a,b),(c,d) select 'yes' from dual where d>a  and c<b;
```

# 分析函数

```sql
select dept_id,min(sale_cnt)keep ( dense_rank first order by sale_date) min_early_date 
from criss_sales where dept_id = 'D02' group by dept_id
```

```
select * ,first_value(s_id) over (partition by c_id order by s_score)first_show from score;
```

例：`RANK() OVER(PARTITION BY DEPTNO ORDER BY SAL DESC) RW `

rank，dense_rank，row_number函数为每条记录产生一个从1开始至n的自然数，n的值可能小于等于记录的总数。这3个函数的唯一区别在于当碰到相同数据时的排名策略。
- row_number： 
row_number函数返回一个唯一的值，当碰到相同数据时，排名按照记录集中记录的顺序依次递增。
- dense_rank： 
dense_rank函数返回一个唯一的值，当碰到相同数据时，此时所有相同数据的排名都是一样的。
- rank： 
rank函数返回一个唯一的值，当碰到相同的数据时，此时所有相同数据的排名是一样的，同时会在最后一条相同记录和下一条不同记录的排名之间空出排名。

# sum(...) over(...)

- sum(…) over( )，对所有行求和

- sum(…) over( order by … )，累计求和

- sum(…) over( partition by… )，同组内所有行求和

- sum(…) over( partition by… order by … )，同组内累计求和

