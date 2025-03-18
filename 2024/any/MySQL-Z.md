MySQL
2022年2月4日
22:20

@不要在这里新增
@See [MySQL](../工程/MySQL.md)


[[toc]]

---
---

最后加  \G   结果表示为列

sql  前面加  explain

通过  MySQL5.5 和 MySQL5.7 的对比，现在大家明白什么是索引下推了吧？其实一句话：在搜索引擎中提前判断对应的搜索条件是否满足，满足了再去回表，通过减少回表次数进而提高查询效率。

like  最左匹配 可以走索引

        对于Mysql来说，如果要做到与业务解耦，首先想到的肯定是binlog，因为binlog毕竟记录了全量真实落库的数据，将自己伪装成一个从节点，订阅binlog就能完成业务上的解耦，

监听binlog。根据curd类型，更新redis

缓存，更新MySQL后，删除redis。而不是更新。

已使用 Microsoft OneNote 2016 创建。


---
---

# explain analyze

https://dev.mysql.com/blog-archive/mysql-explain-analyze/

MySQL 8.0.18 新功能 EXPLAIN ANALYZE
。。不知道和 EXPLAIN 有什么区别。。好像差不多。

EXPLAIN ANALYZE 展示你的 query ，在哪里消耗了哪些时间，及为什么。

这个功能是基于 常规 EXPLAIN 的，可以认为是一种扩展。
除了EXPLAIN的 查询计划和估算消耗 外，还显示执行计划中每个迭代器的实际消耗

有如下sql和结果
```sql
SELECT first_name, last_name, SUM(amount) AS total
FROM staff INNER JOIN payment
  ON staff.staff_id = payment.staff_id
     AND
     payment_date LIKE '2005-08%'
GROUP BY first_name, last_name;

+------------+-----------+----------+
| first_name | last_name | total    |
+------------+-----------+----------+
| Mike       | Hillyer   | 11853.65 |
| Jon        | Stephens  | 12218.48 |
+------------+-----------+----------+
2 rows in set (0,02 sec)
```


`EXPLAIN FORMAT=TREE` 会展示 查询计划 和 估计消耗
```sql
EXPLAIN FORMAT=TREE
SELECT first_name, last_name, SUM(amount) AS total
FROM staff INNER JOIN payment
  ON staff.staff_id = payment.staff_id
     AND
     payment_date LIKE '2005-08%'
GROUP BY first_name, last_name;

-> Table scan on <temporary>
    -> Aggregate using temporary table
        -> Nested loop inner join  (cost=1757.30 rows=1787)
            -> Table scan on staff  (cost=3.20 rows=2)
            -> Filter: (payment.payment_date like '2005-08%')  (cost=117.43 rows=894)
                -> Index lookup on payment using idx_fk_staff_id (staff_id=staff.staff_id)  (cost=117.43 rows=8043)
```

但是没有告诉我们 ==这些估算是否正确== ， 也没有告诉我们 实际消耗。
`EXPLAIN ANALYZE` 可以做到

```sql
EXPLAIN ANALYZE
SELECT first_name, last_name, SUM(amount) AS total
FROM staff INNER JOIN payment
  ON staff.staff_id = payment.staff_id
     AND
     payment_date LIKE '2005-08%'
GROUP BY first_name, last_name;

-> Table scan on <temporary>  (actual time=0.001..0.001 rows=2 loops=1)
    -> Aggregate using temporary table  (actual time=58.104..58.104 rows=2 loops=1)
        -> Nested loop inner join  (cost=1757.30 rows=1787) (actual time=0.816..46.135 rows=5687 loops=1)
            -> Table scan on staff  (cost=3.20 rows=2) (actual time=0.047..0.051 rows=2 loops=1)
            -> Filter: (payment.payment_date like '2005-08%')  (cost=117.43 rows=894) (actual time=0.464..22.767 rows=2844 loops=2)
                -> Index lookup on payment using idx_fk_staff_id (staff_id=staff.staff_id)  (cost=117.43 rows=8043) (actual time=0.450..19.988 rows=8024 loops=2)
```

包含了一些新的测量值
- 获得第一行的 实际毫秒
- 获得所有行的 实际毫秒
- 读取的 实际行数
- 循环的 实际次数

---

针对下面的输出 进行说明
```text
Filter: (payment.payment_date like '2005-08%')
(cost=117.43 rows=894)
(actual time=0.464..22.767 rows=2844 loops=2)
```

预估： 消耗117.43ms，返回894行。 这些预估是 在真正执行前， 由 查询优化器 估计的。

我们从后往前看，loop2的值是2，要理解这个数字，我们要看 查询计划的 上下文，
在第11行，有一个 `Nested loop inner join` ，
第12行，有一个 `Table scan`
这就意味着，我们 进行了一个 nested loop ， 来扫描 staff 表，我们使用 `Index Lookup`  在 payment 表中 搜索 对应的 数据，并且 对 payment数据 进行了 `Filter`。由于staff 表 有2行，所以要进行2次 loop 来根据 index 搜索数据，所以是2
。Since there are two rows in the staff table (Mike and Jon), we get two loop iterations on the filtering, and on the index lookup on line 14.

`actual time=0.464..22.767` ， 意味着 读到 第一行的耗时是 0.464ms，读取所有行的 耗时是 22.767ms。 这个是 ==平均时间==， 因为 我们执行了 2次 loop，所以 会返回 所有loop 的 平均时间。 这意味着 filter 的实际执行时间 需要 * 2。 所以我们可以看到 ==第11行==的 `Nested loop` 的最后 `actual time=0.816..46.135`  ，==总耗时==是 46.135ms， 是 22.767ms 的 2倍 多一点点。

最后一行 `Index lookup` ，显示 0.450 和 19.988ms，这意味着 大部分的时间都是 用于 通过 index lookup 读取 row， 实际的filter 用时 较少。

读取的行数 是 2844， 预估是 894。预估失真了。 这里的值也是 平均值。
在 payment_data 列上，没有 index 或 直方图，所以 提供给 优化器 的统计信息是 有限的。 (。。所以导致预估失真)。 
可以看 最后一行，`Index lookup` ， 实际 8024，预估 8043， 非常接近。 这是因为 index 附带了一些统计信息。

分析和理解 为什么这样执行，需要一些练习，这里有一些 简单的 hint
- 如果你想知道 为什么 消耗那么长的时间，那么看 timing，查看 哪里消耗最多
- 如果你想知道 优化器为什么 选择这个plain，请查看 row counter。 预估的行数 和 实际行数 有很大差异，说明你应该 仔细观察它。 优化器根据 估计值 选择计划， 但 实际执行情况，可能告诉你，另一个计划 更好。

---

- 要检查 查询计划：`EXPLAIN FORMAT=TREE`
- 要看到 查询执行：`EXPLAIN ANALYZE`
- 要理解 plan 选择：optimizer trace



# explain

https://baike.baidu.com/item/explain%20mysql/4538417?fr=ge_ala

```sql
EXPLAIN SELECT * FROM `TABLE_NAME`;
```

|id|select_type|table|type|possible_keys|key|key_len|ref|rows|Extra|
|--|--|--|--|--|--|--|--|--|--|
|1|SIMPLE|table_name|ALL|NULL|NULL|NULL|NULL|100|-|

- table列，显示 sql语句 涉及的表
- type列，很重要，显示 join 使用了哪种类型，有没有使用index，反映了语句的质量
- possible_keys列，指出 MySQL 能使用 哪个index 在该表中找到 行
- key_len列，显示MySQL决定使用的key长度。 在不损失精确性的情况下， 长度 越短越好。
- ref，使用哪个列或常数 与 key 一起 从表中选择行
- rows，mysql 认为 它执行查询时必须检查的行数
- extra，mysql解决查询的 详细信息

explain的 type 显示的是访问类型，是比较重要的一个指标，从好到怀依次是
`system ＞ const ＞ eq_ref ＞ ref ＞ fulltext ＞ ref_or_null ＞ index_merge ＞ unique_subquery ＞ index_subquery ＞ range ＞ index ＞ ALL`

一般，至少保证达到 range，最好能达到 ref。

---

`EXPLAIN SELECT * FROM TABLE_NAME WHERE id=1;`

|id|select_type|table|type|possible_keys|key|key_len|ref|rows|Extra|
|--|--|--|--|--|--|--|--|--|--|
|1|SIMPLE|table_name|const|primary|primary|4|const|1|-|


---
`EXPLAIN SELECT * FROM (SELECT * FROM (SELECT * FROM TABLE_NAME WHERE id<100) a) b`

|id|select_type|table|type|possible_keys|key|key_len|ref|rows|Extra|
|--|--|--|--|--|--|--|--|--|--|
|1|SIMPLE|derived2|ALL|NULL|NULL|NULL|NULL|99|-|
|2|DERIVED|derived3|ALL|NULL|NULL|NULL|NULL|99|-|
|3|DERIVED|table_name|range|primary|primary|4|NULL|102|-|


---

## select_type 
分为
- SIMPLE，简单select，没有 union 或子查询
- PRIMARY，最外层的select
- UNION，union中第二个 或 后面的 select
- DEPENDENT UNION，union中第二个 或 后面的 select，取决于外面的查询
- UNION RESULT，union的结果
- SUBQUERY，子查询中第一个select
- DEPENDENT SUBQUERY，子查询中第一个select 取决于外面的查询
- DERIVED，派生表的select (from子句的 子查询)


## table
这行的数据是关于 哪张表的

## type
连接 使用了哪种类别
- system，
  const连接类型的一个特例。 表仅有一行满足条件
- const，
  最多有一个匹配行。
- eq_ref，
  对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY
- ref
  对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref
- ref_or_null
  该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化
- index_merge
  该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素
- unique_subquery
  该类型替换了下面形式的IN子查询的ref：`value IN (SELECT primary_key FROM single_table WHERE some_expr)`
- index_subquery
  该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引 `value IN (SELECT key_column FROM single_table WHERE some_expr)`
- range
  只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。
- index
  该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。当查询只使用作为单索引一部分的列时，MySQL可以使用该联接类型。
- all
  对于每个来自于先前的表的行组合，进行完整的表扫描。

## possible_key

## key
MySQL实际决定使用的键（索引）
要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用`FORCE INDEX、USE INDEX或者IGNORE INDEX`。

## key_len
显示MySQL决定使用的键长度
在不损失精确性的情况下，长度越短越好

## ref
使用哪个列或常数与key一起从表中选择行。

## row
MySQL认为它执行查询时必须检查的行数。

## Extra
MySQL解决查询的详细信息
- Distinct
  一旦找到 匹配的行，就不再搜索
- Not exist
  mysql优化了 left join，一旦找到 匹配 left join 的行，就不再搜索
- range checked for each
  没有找到理想的index，因为 对于从前面表中来的 每一个行组合，mysql检查使用哪个index，它来从 表中 返回行。这是 使用index 的 最慢的连接之一。
- using filesort
  看到这个，查询就需要优化。 mysql需要额外的步骤来发现如何 对返回的行排序。它根据 连接类型 以及 存储的 排序键值 和 匹配条件 的全部行的 行指针 来排序 全部行
- using index
  列数据是从 仅仅使用了 索引中的信息 而没有读取实际的行动的表返回的。这发生在对表的全部的请求列都是同一个索引的部分的时候
- using temporary
  查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上
- using where
  使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题



---

# explain

https://blog.csdn.net/m0_67695717/article/details/130392997


explain 有2个变种
- explain extended
  告诉服务器“逆向编译”执行计划为一个select语句。可以通过紧接其后运行showwarnings看到这个生成的语句。这个语句直接来自执行计划，而不是原SQL语句
- explain partitions
  显示查询将访问的分区，如果查询是基于分区表的话。


id列
数字越大 越先执行。 如果一样大，就 从上往下 执行。
null说明是 结果集，不需要使用它来进行查询。

select_type列
对应行是 简单 还是 复杂 select
常见有
- simple
  表示不包含union操作或者不包含子查询的简单select查询。有连接查询时，外层的查询为simple，且只有一个
- primary
  一个需要union操作或者含有子查询的select，位于最外层的单位查询的select_type即为primary。且只有一个
- union
  union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union
- depentdent union
  与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响
- union result
  包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null
- subquery
  除了from字句中包含的子查询外，其他地方出现的子查询都可能是subquery
- dependent subquery
  与dependentunion类似，表示这个subquery的查询要受到外部表查询的影响
- derived
  from字句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select


table列
对应行正在访问查询的 表名
如果查询使用了别名，那么这里显示的是别名，
如果不涉及对数据表的操作，那么这显示为null，
如果显示为 尖括号 括起来的就表示这个是 临时表，后边的N就是执行计划中的id，表示 结果来自于这个查询产生。
如果是尖括号括起来的`<union M,N>`，与类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。


type列
访问类型,即MySQL决定如何查找表中的行。

依次从好到差：
`system，const，eq_ref，ref，fulltext，ref_or_null，unique_subquery，index_subquery，range，index_merge，index，ALL`
除了all之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引

- system
  表中只有一行数据或者是空表，且只能用于myisam和memory表。如果是Innodb引擎表，type列在这个情况通常都是all或者index
- const
  使用唯一索引或者主键，返回记录一定是1行记录的等值where条件时，通常type是const。其他数据库也叫做唯一索引扫描
- eq_ref
  出现在要连接过个表的查询计划中，驱动表只返回一行数据，且这行数据是第二个表的主键或者唯一索引，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref
- ref
  不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找。或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现。
- fulltext
  全文索引检索，要注意，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引
- ref_or_null
  与ref方法类似，只是增加了null值的比较。实际用的不多。
- unique_subquery
  用于where中的in形式子查询，子查询返回不重复值唯一值
- index_subquery
  用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。
- range
  索引范围扫描，常见于使用>,< ,isnull,between ,in ,like等运算符的查询中。
- index_merge
  表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所个索引，性能可能大部分时间都不如range
- index
  索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询。
- all
  这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录。


possible_keys
查询可能使用到的索引都会在这里列出来。这个列表是优化过程早期创建的，因此有些罗列出来的索引有可能后续是没用的。


key列
显示了查询真正使用到的索引，
select_type为index_merge时，这里可能出现两个以上的索引，
其他的select_type这里只会出现一个。
如果该索引没有出现在possible_keys列中，那么MySQL选用它是出于另外的原因，比如选择了一个覆盖索引。
possible_keys揭示了哪一个索引能有助于高效地行查找，key显示了优化采用哪一个索引可以最小化查询成本。


key_len列
用于处理查询的索引长度，
如果是单列索引，那就整个索引长度算进去，
如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去。
留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。要注意，mysql的ICP特性使用到的索引不会计入其中。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中。


ref列
如果是使用的常数等值查询，这里会显示const，
如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，
如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func


rows列
这里是执行计划中估算的扫描行数，不是精确值


extra列
这个列可以显示的信息非常多，有几十种，常用的有
- distinct
  select部分使用了 distinct 关键字
- no tables used
  不带from字句的查询或者 from dual查询
- using filesort
  排序 时无法使用到索引时，就会出现这个。常见于order by和group by语句中
- using index
  查询时不需要回表查询，直接通过索引就可以获取查询的数据。
- using join buffer (block nestedloop), using join buffer (batched key access)
  5.6.x之后的版本优化关联查询的BNL，BKA特性。主要是减少内表的循环数量以及比较顺序地扫描查询。
- using sort_union, using union, using intersect, using sort_intersection
  using intersect：表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集
  using union：表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集
  using sort_union和usingsort_intersection：与前面两个对应的类似，只是他们是出现在用and和or查询信息量大时，先查询主键，然后进行排序合并后，才能读取记录并返回。
- using intersect
  表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来。
- using where
  表示存储引擎返回的记录并不是所有的都满足查询条件，需要在server层进行过滤。
- firstmatch(tb_name)：
  5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个
- loosescan(m…n)：
  5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个


filtered列
使用explain extended时会出现这个列，5.7之后的版本默认就有这个字段，不需要使用explain extended了。
这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数。







---

# MySQL 优化 15个技巧

bilibili

慢sql的优化，从2个方面考虑，
- sql语句本身的优化
- 数据库设计的优化

## 避免不必要的列

sql查询的时候，只查询需要的列。

## 分页优化
数据量比较大，分页比较深的情况下，考虑分页的优化
深分页
`select * from table where type=2 and level=9 order by id asc limit 100000,10;`

- 延迟关联
先通过 where 搜索出 主键，然后 再与 原数据表 关联，通过 主键 提取数据行，而不是 通过原来的 二级索引提取数据行
`select * from table a, (select id from table where type=2 and level=9 order by id asc limit 100000,10) b where a.id=b.id`
。。？ 这条sql 可以？ 感觉没什么用吧。 而且 似乎错了？ b是集合 怎么 =b.id？  

- id偏移量
找到limit 第一个参数 所对应的主键，然后通过 这个 主键 去过滤并 limit
`select * from table where id > (select * from table where type=2 and level=9 order by id asc limit 190);`
。。。感觉纯粹胡扯啊。。

## 索引优化
- 利用覆盖索引
  查询列 都是 索引列
  减少回表
- 避免 or查询
  使用 union 或 子查询 来替代。
  早期mysql 使用 or 可能会导致 索引失效，高版本 加入了 ==索引合并==，解决了这个问题。
- 避免 `!=, <>` 操作
  转为 or， a!=123 -> `a < 123 or a > 123`
- 适当使用 前缀索引
  减少 索引的空间占用，提高索引的查询效率。 比如 邮箱，后面是 比较 固定的 @xx.com 。比较冗余。。。不过 前面的长度不是固定的啊。
  mysql无法使用 前缀索引 进行 order by, group by, 也无法 覆盖索引
- 避免列上函数运算
- 正确使用联合索引
  注意最左匹配


## join优化

- 优化子查询
  使用join 代替子查询
- 小表驱动大表
  减少表连接 创建次数  。。 我以为是 mlogn 和 nlogm 的区别。
- 适当增加冗余字段
  不需要join就可以得到数据
- 避免join太多的表


## 排序优化
- 利用索引扫描作排序
  利用 索引覆盖


## union优化
- 条件下推
  mysql处理 union的策略是 先创建 临时表，然后将 各个查询结果填充到 临时表，最后进行查询。
  将 where，limit 等子句 下推到 union 的各个子查询中。
- 使用 union all
  不带all， 临时表是 distinct的， 代价很大。



# 自增id在批量插入时，每次申请1,2,4,8个id
不知道上限是多少个。

























