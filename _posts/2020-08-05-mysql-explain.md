---
layout: post
title: "Mysql执行计划详解"
subtitle: ""
date: 2020-08-05 15:50:34 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - Mysql 
tags:
    - Mysql   
---


#### id
  * 执行编号，标识select所属的行。
  * 如果在语句中没子查询或关联查询，只有唯一的select，每行都将显示1。否则，内层的select语句一般会顺序编号，对应于其在原始语句中的位置, 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

#### select_type
  * 显示本行是简单或复杂select。如果查询有任何复杂的子查询，则最外层标记为PRIMARY（DERIVED、UNION、UNION RESUlT）
  1. SIMPLE (简单SELECT,不使用UNION或子查询等)
  2. PRIMARY (查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
  3. UNION (UNION中的第二个或后面的SELECT语句)
  4. DEPENDENT UNION (UNION 中的第二个或后一个 SELECT语句，具体取决于外部查询)
  5. UNION RESULT (UNION 结果集)
  6. SUBQUERY (子查询中的第一个SELECT)
  7. DEPENDENT SUBQUERY (子查询中的第一个 SELECT，取决于外部查询)
  8. DERIVED (派生表的SELECT, FROM子句的子查询)
  9. DEPENDENT DERIVED (依赖于另一个表的派生表)
  10. MATERIALIZED (将子查询作为临时表)
  11. UNCACHEABLE SUBQUERY (无法缓存结果的子查询，必须为外部查询的每一行重新评估)
  12. UNCACHEABLE UNION (属于不可缓存子查询的UNION中的第二个或更晚选择)
 
#### table
  * 对应行正在访问哪一个表，表名或者别名,这也可以是以下值之一
  1. <unionM,N>：行是指值为 M 和 N 的行的联合。
  2. <derivedN>：该行引用值为 N 的行的派生表结果。例如，派生表可能来自子查询。
  3. <subqueryN>：该行是指值为 N 的行的物化子查询(临时表)的结果
  
#### partitions
  * 5.7以后成为了默认选项。该列显示的为分区表命中的分区情况。非分区表该字段为空  

#### type
  * 访问类型,结果值从好到坏依次是：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL ，一般来说，得保证查询至少达到range级别，最好能达到ref。
  1. system (这是const连接类型的一种特例，表仅有一行满足条件)
  2. const (该表最多有一个匹配的行，在查询开始时读取该行。由于只有一行，因此优化器的其余部分可以将该行中列中的值视为常量。 const表非常快，因为它们只读一次。)
  3. eq_ref (类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件)
  4. ref (一种索引访问，它返回所有匹配某个单个值的行。此类索引访问只有当使用非唯一性索引或唯一性索引非唯一性前缀时才会发生。这个类型跟eq_ref不同的是，它用在关联操作只使用了索引的最左前缀，或者索引不是UNIQUE和PRIMARY KEY。ref可以用于使用=或<=>操作符的带索引的列。)
  5. fulltext (全文索引)
  6. ref_or_null (该类型和ref类似。但是MySQL会做一个额外的搜索包含NULL列的操作。在解决子查询中经常使用该联接类型的优化)
  7. index_merge (此联接类型指示使用索引合并优化。在这种情况下，输出行中的列包含使用的索引列表，并包含使用索引的最长键部分的列表)
  8. unique_subquery (一个索引查找函数，它完全替换子查询，以提高效率)
  ```sql
value IN (SELECT primary_key FROM single_table WHERE some_expr)  
  ```
  9. index_subquery (该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引)
  ```sql
  value IN (SELECT key_column FROM single_table WHERE some_expr)
  ```
  10. range (只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range)
  11. index (联接类型与ALL相同，只是扫描索引树。这发生两种方式)
  - 如果索引是查询的覆盖索引，并且可用于满足表中所需的所有数据，则仅扫描索引树。在这种情况下，列中显示 。仅索引扫描通常比ALL 快，因为索引的大小通常小于表数据。
  - 用从索引读取来按索引顺序查找数据行，执行完整的表扫描。 不显示在列中
  12. ALL (最慢的一种方式，全表扫描)

#### possible_keys
  * 显示查询使用了哪些索引，表示该索引可以进行高效地查找，但是列出来的索引对于后续优化过程可能是没有用的
 
#### key
  * MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL
  
#### key_len
  * 显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好 。

#### ref
  * 该列显示哪些列或常量与列中命名的索引进行比较，以从表中选择行
  
#### rows
  * 该列指示 MySQL认为必须检查才能执行查询的行数,估算的找到所需的记录所需要读取的行数 

#### filtered
  * 该列指示将由表条件筛选的表行的估计百分比。最大值为 100，这意味着未对行进行筛选。
  
#### Extra
  * 性能从好到坏:useing index>usinh where > using temporary > using filesort
  1. distinct (在select部分使用了distinc关键字)
  2. no tables used (不带from字句的查询或者From dual查询。 使用not in()形式子查询或not exists运算符的连接查询，这种叫做反连接。即，一般连接查询是先查询内表，再查询外表，反连接就是先查询外表，再查询内表)
  3. using filesort (排序时无法使用到索引时，就会出现这个。常见于order by和group by语句中)
  4. using index (查询时不需要回表查询，直接通过索引就可以获取查询的数据)
  5. using_union (表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集)
  6. using intersect (表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集)
  7. using sort_union和using sort_intersection (与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回)
  8. using where (表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server层进行过滤。查询条件中分为限制条件和检查条件，5.6之前，存储引擎只能根据限制条件扫描数据并返回，然后server层根据检查条件进行过滤再返回真正符合查询的数据。5.6.x之后支持ICP特性，可以把检查条件也下推到存储引擎层，不符合检查条件和限制条件的数据，直接不读取，这样就大大减少了存储引擎扫描的记录数量。extra列显示using index condition)
  9. using temporary (表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来)
  10. firstmatch(tb_name) (5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个)
  11. loosescan(m..n) (5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个)
  12. filtered (使用explain extended时会出现这个列，5.7之后的版本默认就有这个字段，不需要使用explain extended了。这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数)
  
  
  
  
  
  
  
  
  
  
