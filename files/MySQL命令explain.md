---
title: MySQL命令explain
date: 2022-06-04 18:12:00
tags:
  - 数据库
  - MySQL
  - 优化
---

`explain`是MySQL查询优化的关键命令. 通过这个命令可以模拟优化器执行SQL查询语句, 分析执行结果可以优化查询语句以提升查询性能.

<!--more-->

语句:

```sql
explain select * from xx where yy = 'zz'
```

相应字段介绍:

- id : 语句中select查询子句的顺序

- select_type : 查询类型

  - SIMPLE : 简单查询, 不包含子查询与union查询

  - PRIMARY : 主查询, 存在子查询时,外层查询被标记为主查询

  - SUBQUERY : 子查询

  - UNION : 在union命令后的语句为此类型

  - UNION RESULT : 最终UNION结果会表示为此类型

    ```
        id  select_type   table       partitions  type    possible_keys        key      
    ------  ------------  ----------  ----------  ------  -------------------  ------- 
         1  PRIMARY       student     (NULL)      const   PRIMARY,id_name_age  PRIMARY  
         2  UNION         student     (NULL)      const   PRIMARY,id_name_age  PRIMARY 
    (NULL)  UNION RESULT  <union1,2>  (NULL)      ALL     (NULL)               (NULL)   
    ```

  - DERIVED : FROM列表中包含子查询的语句被标记为此类型, MySQL会递归执行这些子查询放入临时表(MySQL 5.7+ 优化后使用derived_merge加速查询效率. 无此状态)

- table : 访问的表,结合id可以看到执行过程中表查询顺序

- partitions : 匹配分区

- type : 访问类型

  10个状态,从好到差排序:

  ```
  NULL > system > const > eq_ref > ref > ref_or_null > index_merge > range > index > ALL
  ```

  - NULL : 无需访问表和索引的查询. 例: 

    ```sql
    EXPLAIN SELECT 5*7  -- 不访问表,计算数值
    EXPLAIN SELECT MAX(id) FROM student  -- 查询最大索引,直接取索引树叶子结点获取 
    ```

  - SYSTEM : 表只有一行记录（等于系统表），这是`const`类型的特列，平时不大会出现，可以忽略。

  - const : 一次索引查找到,主键索引或唯一索引查询时

  - eq_ref : 联表join查询时, 按联表的主键或唯一键查询

  - ref : 联表join查询时, 按联表的索引查询, 匹配多行不唯一

  - ref_or_null : 类似ref, 可以搜索值为NULL的行

  - index_merge : 查询了多个索引,然后取交集并集. 常见于在`and`, 'or'的条件下使用了不同的索引. 大部分性能不如`range`

  - range : 索引范围查询,常见于 <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN()或者like等运算符的查询中

  - ALL : 全表扫描, 务必优化!!!

- possible_keys : 可能使用的索引

- key : 实际使用的索引, 为null时表示没使用索引

- key_len : 索引中使用的字节数

- ref : 索引的哪一列被使用

- rows : 根据表统计信息以及索引选用情况,估算查找所需记录需要读取的行数. 数值越小效率越高

- filtere : 查询数据占表的百分比. 类似rows,数值越小效率越高

- extra : 其他重要的额外信息

  - Using filesort : SQL需要排序, 但是优化器找不到可以使用的索引, 此时使用外部排序(多次磁盘IO访问,效率极低)
  - Using tempporary : 查询结果排序时, 使用临时表协助. 效率低于外部排序.
  - Using index : 使用了索引 👍🏻👍🏻👍🏻
  - Using where : 使用了where
  - Using join buffer : 多表join使用了连接缓存
  - impossible where : 筛选条件啥也没找到
  - distinct : 优化distinct操作, 找到匹配后立即停止同样值的动作
