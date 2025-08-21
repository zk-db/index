---
title: Sql 面试
date: 2022-06-04 16:49:30
tags:
  - sql
  - 面试
---

1. having

   having常用语分组后的条件过滤常用于`group by xx `后

   例子:

   ```sql
   SELECT column_name, aggregate_function(column_name)
   FROM table_name
   WHERE column_name operator value
   GROUP BY column_name
   HAVING aggregate_function(column_name) operator value
   ```

   注意事项:

   - having 子句内容需要存在select列表中
   - having限制的是组, where限制的是行
   - where, group by, having执行顺序: where过滤行 -> group by对数据分组,并执行聚集函数 -> having过滤符合条件的组

   <!--more-->

2. `select a.xx,b.xx from a,b where a.x = b.x`与`select a.xx,b.xx from a inner join b on a.x = b.x` 区别

3. union与union all

   union会去重, union all 直接合并

4. row_number()

   ```sql
   --在test表中根据name分组，age进行排序
   select name,age,row_number() over(partition by name order by
   age desc) from test;
   ```

   ```sql
   --去掉重复的记录
   select * from (select name,age,row_number() over( partition by name 
   order by age desc) rn from test )where rn= 1;
   ```

   可以对结果集进行分组排序并生成一个row_number序列

5. DDL, DML与DCL

   - DDL: Data Define Language, 数据定义语句. CREATE, ALTER, DROP等
   - DML: Data Manage Language, 数据管理语句. SELECT, UPDATE, INSERT, DELETE
   - DCL: Data Control Language, 数据控制语句. GRANT, DENY, REVOKE等个更新用户角色的命令

6. exists 与 in

   - `in` 仅执行一次(先查询子查询的表，然后将内表和外表做一个笛卡尔积，然后按照条件进行筛选), `exists`执行多次(外表行数)
   - 外层查询小于子查询使用`exists`; 外层查询大于子查询使用`in`
   - `in` 与`exists`都会使用索引
   - `not in` 内外表均全局扫描,无法使用索引, `not exists`可以走索引
   - `in`的遍历实在内存中,速度较快. `exists`多次查询数据库. 

7. 索引使用注意事项

   - 尽量全值匹配
   - 模糊查询使用最左匹配原则
   - 全模糊查询使用覆盖索引方式也会走索引
   - 不在索引列上进行操作
   - 联合索引需要严格顺序匹配
   - 尽量使用覆盖索引
   - mysql在使用`!=`与`<>`时索引失效
   - 字符串不加单引号导致索引失效
   - `or`使得索引失效

8. 索引创建

   - 频繁查询的字段应创建索引
   - 与其他表关联查询的字段创建索引
   - 频繁更新的字段不适合作为索引

9. sql书写题目

   参考: [经典题目](https://blog.csdn.net/jason_wang1989/article/details/115278576)

