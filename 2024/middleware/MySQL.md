

MySQL
2022年2月16日
8:55

[[toc]]

=====================

# mysql调优

https://www.cnblogs.com/zhanxiaomi/p/14959059.html

调优的重点在于：开发规范，数据库索引，解决线上慢查询问题。

如果有一定的数据量，就应该创建对应的索引。

查询需要注意的地方

1.  能否使用 索引覆盖，减少 回表 消耗的时间，就是说，我们查询时一定要指明查询的列，而不是select *
    
2.  考虑是否组建 联合索引，尽量将区分度最高的放在最左边，并考虑 最左匹配原则
    
3.  避免对索引进行 函数操作会表达式计算，因为会导致 索引失效
    
4.  通过 explain 命令查看 sql 的执行计划，查看是否走了索引，走了什么索引，通过 show profile 来看 sql对系统资源消耗的情况
    
5.  开启事务后，在 事务内，尽可能只操作数据库，并有意识地 减少锁的持有时间(比如在事务内插入和修改数据，可以先插入再修改，因为修改是更新操作，会加行锁，如果先更新，在并发场景下回导致多个事务的请求在等待行锁的释放。。。？insert，update执行顺序都是update的时候行锁啊，有什么区别)
    

走了索引，但线上查询慢。

走了索引还慢，基本是因为表的数据量太大了，可以考虑删除 旧数据，但一般不会这么做。那么就可以考虑 走一层缓存(redis)，这时需要考虑业务能否接受非真正实时的数据，而如果查询条件比较复杂，那么缓存也不是一种好的办法

再往后，就是查看 是否有字符串检索的场景，导致查询效率低，如果有，可以考虑把数据导入Elasticsearch。
还不行，就考虑查询条件的维度，做一些聚合表，线上请求直接查询聚合表。
还有读写分离，分库分表。

=====================

=====================

=====================

# 从源码看MySQL的加锁规则

https://juejin.cn/post/7133014068830404622

InnoDB 使用 lock_t 这个结构来定义：

![1](../_resources/a6ef6b9d35514d26b54376c37d2e3da8.jpg)

对一条记录 加锁 的本质 就是 在内存中 创建一个 锁结构 与之关联。
那么是不是 一个事务 对多条记录加锁，就要创建多个锁结构呢？ 如果一个事务要 10000条记录的锁，就要 10000个锁，开销太大。

InnoDB 在对不同记录加锁时，如果符合下面的条件，那么这些记录的锁 可以被放到 一个 锁结构中：
在同一个事务中进行加锁
被加锁的记录在同一个页面中
加锁的类型是一样的
等待状态是一样的

锁内存结构示意图

![1](../_resources/adcd1659c6064715849b3cad304afa5b.jpg)

锁所在事务信息：哪个事务生成了这个锁结构，就记载这个事务的信息
索引信息：对于行锁来说，需要记录一下加锁的记录是属于哪个索引的
表锁/行锁信息：表锁结构和行锁结构在这个位置的内容是不同的：
表锁：记载着这是对哪个表加的锁，还有其他的一些信息
行锁：记载着 3个重要信息：
Space ID：记录所在表空间
Page Number: 记录所在页号

n\_bits：对于行锁来说，一条记录 就对应着一个bit位，一个页面中包含很多记录，用不同的bit位 来区分到底是 哪一条记录加了锁。为此在行锁结构的末尾放置了一堆bit位，这个n\_bits 属性 代表 使用了多少bit位。

type\_mode: 32位int，被分成 lock\_mode, lock\_type, rec\_lock_type 3部分
lock_mode
占用低 4位，可选值：
LOCK_IS，十进制0，表示 共享意向锁，也就是 IS锁
LOCK_IX，十进制1，表示 独占意向锁，也就是 IX锁
LOCK_S，十进制2，共享锁，即S锁
LOCK_X，十进制3，独占锁，即X锁
LOCK\_AUTO\_INC，十进制4，表示 AUTO-INC 锁
lock_type
占用低5-8位，目前只有 低5,6位 被使用：
LOCK_TABLE, 十进制16，即第5个bit为1，表级锁。
LOCK_REC, 十进制32，即第6个bit为1，行级锁。
rec\_lock\_type
行级锁的 具体类型，使用其余的bit来表示，只有在 lock\_type 为 LOCK\_REC时，才会被细分出更多类型:
LOCK_ORDINARY, 0 (。。不是256？)，表示 next-key 锁
LOCK_GAP, 512，也就是当 第10个bit位 为1时，表示gap 锁
LOCK\_REC\_NOT_GAP, 1025，11个bit为1，表示 行锁
LOCK\_INSERT\_INTENTION, 2048， 表示 插入意向锁。
其他的类型
其他信息：
为了更好管理 系统运行过程中 生成的各种 锁结构 而设计了各种 哈希表 和链表
比特位：

如果是 行锁 结构的话，在该结构的末尾 还放置了 一堆 bit。页面中的每条记录在记录头信息中 都包含一个 heap\_no 属性，下界(Infinum) 的heap\_no 为0，上界(supremum)的heap\_no为1，之后每插入 一条记录，heap\_no 值就+1。 锁结构最后的一堆比特位 就对应着一个页面中的记录，一个bit位 映射为 一个 heap_no

。。。？？？

。。RECord ？

InnoDB的实现中，InnoDB的行锁 是与 记录 一一对应的，对于 间隙锁(GAP) 来说也是如此，虽然说 间隙锁 锁住的是一个 间隙，但是通过在 指定记录上 加锁 生成 锁结构，然后指定锁的类型 为 间隙锁，并不是 在一个专门的 区间生成一个间隙锁。它的工作方式就是在 插入记录时 检查下一条记录上的 锁结构是否 是间隙锁，是的话就无法插入。

## 加锁受哪些因素影响

事务的隔离级别
语句执行时 使用的索引类型 (如，聚簇索引，唯一二级索引，普通二级索引)
是否精确匹配
是否唯一性搜索
具体执行的语句类型(select,insert,delete,update)
记录是否被标记删除
是否开启innodb\_locks\_unsafe\_for\_binlog系统变量

innodb\_locks\_unsafe\_for\_binlog最主要的作用就是控制innodb是否对gap加锁。注意该参数如果是enable的，则是unsafe的，此时gap不会加锁；反之，如果disable掉该参数，则gap会加锁

扫描区间
select * from hero where name<='1XXX';
MySQL 可以有2种方式 执行上面的查询：

使用二级索引 idx_name 执行上面的查询，那么就需要扫描 name 在 (-inf,'1XXX'\] 区间的 所有二级索引指针，针对获取到的 每条 二级索引，都需要执行 回表操作 来获得对应的 聚簇索引记录

直接扫描所有的 聚簇索引记录，即进行全表扫描。此时相当于扫描(-inf, +inf) 区间中 所有 聚簇索引记录

可能有人有疑问，name 不是二级索引吗？ 为什么还有可能 不走索引 而去 全表扫描呢？

SQL 的执行成本（cost）是 MySQL 优化器选择 SQL 执行计划时一个重要考量因素。当优化器认为使用索引的成本高于全表扫描的时候，优化器将会选择全表扫描，而不是使用索引。那为什么使用索引的成本比全表扫描还高呢？因为当普通索引并不包括查询的所有列（没有使用覆盖索引的情况），因此需要通过 name 的索引树找到对应的主键 id ，然后再到 id 的索引树进行数据查询，即回表（通过索引查出主键，再去查数据行），这样成本必然上升。尤其是当回表的数据量比较大的时候，经常会出现 MySQL 优化器认为回表查询代价过高而不选择索引的情况。

精确匹配
如果是 等值匹配的 条件，我们就把这个扫描区间 的匹配模式 称为 精确匹配
select * from hero where name='1xxx' and country='qq'
如果使用二级索引 idx_name 执行上面的查询，扫描区间就是 \['1xxx','1xxx'\]。

唯一性搜索

如果扫描某个区间 前， 就能确定 该扫描区间 最多只包含 1条记录的话，那么就把 这种情况称为 唯一性搜索。那么 满足什么样的情况 可以确定 是 唯一性搜索呢？

1.  匹配模式 是精确匹配
2.  使用的索引是 聚簇索引 或唯一索引(非聚簇索引)
3.  如果索引中包含多个列，则每个列在生成 扫描区间时 都应该被用到。

比如，为表的 a，b 两列 建立一个 唯一索引 uniquek\_a\_b(a,b)，那么对于 搜索条件a=1形成的 扫描区间来说，不能保证 该扫描区间最多包含 1条数据；对于a=1 and b=1 形成的扫描区间来说， 才能保证 扫描区间只有 1条数据。

1.  如果使用的索引是唯一索引(非聚簇索引)，那么搜索时 不能搜索某个 索引列 为null的记录 (因为对于 唯一索引来说，是可以存储多个值为null的记录的)

row\_search\_mvcc

MySQL其实是分为 server 层 和存储引擎层 2部分。每当执行一个 查询时，server 层负责 生成 执行计划，即选取 即将使用的 索引 以及对应的 扫描区间。我们以 InnoDB 为例，针对每个 扫描区间，都会：

1.  server 层向 InnoDB 索要 扫描区间的 第一条记录
    
2.  InnoDB 通过B+ 树 定位到 扫描区间的 第一条记录，(如果定位的是 二级索引 并有 回表需求 则 回表 获取完整的 聚簇索引记录)，然后返回给 server 层
    
3.  server层 判断 记录是否 符合搜索条件，如果符合则发送给客户端，不符合则跳过。 继续向 InnoDB 索要 下一条记录
    

此处将记录发给给客户端 实际上是 发送到本地的 网络缓冲区，缓冲区的 大小 由 net\_beffer\_length 控制，默认 16kb。 等缓冲区满了，才真正发送网络包到客户端。

1.  InnoDB 根据记录 的 单向链表 以及 页面之间的 双向链表 找到 下一条记录(如果定位的是 二级索引 并 有回表需求，则回表获得 完整的 聚簇索引记录)，返回给server层
    
2.  server层处理该记录，并向 InnoDB 索要 下一条记录
    
3.  不停执行上述过程，知道 InnoDB 读到一条不符合 边界条件的 记录为止
    

可见，一般情况下，server 层 和 存储引擎层 是以 记录为单位 进行通信的，而 InnoDB 读取 一条记录 最重要的函数 就是 row\_search\_mvcc， 在这个函数中，对一条记录进行 诸如 多版本可见性判断，是否对记录加锁的判断，加什么锁，完成 记录 从 InnoDB 的存储格式 到server 层 存储格式的转换 等等。

## 语句到底是怎么加锁的

https://juejin.cn/post/7133014068830404622
下面都有代码截图的。。。

对普通的select的处理 和意向锁的添加

对于Repeatable Read 隔离级别来说， 只在首次执行 select 时 生成 Readview，之后的 select 都会复用这个 ReadView。

对于Read Committed 隔离级别来说，每次执行 select 都会生成一个 ReadView。

定位扫描区间的第一条记录

对于order by xxx desc 条件形成的 扫描区间的 第一条记录的处理

对于infimum，supremum 记录的处理

对于精确匹配的特殊处理

真正的加锁流程才开始

判断 索引条件下推 的条件是否成立
Index Condition Pushdown， ICP

回表对记录加锁

row\_search\_mvcc 返回，判断是否已经到达边界

然后，再处理下一条记录。

=====================

=====================

# 索引

聚簇索引：聚簇顾名思义，聚集在一起，即索引和数据是存放同一个文件中。其叶子节点中存放的就是整张表的行记录数据，也将聚集索引的叶子节点称为数据页。InnoDB引擎使用的是非聚簇索引。

。。主键是聚簇，其他非聚簇(叶子节点保存主键，需要二次回表)吧？

非聚簇索引：索引文件和数据文件是分开的。MyISAM引擎默认使用的是非聚簇索引。

1.innodb如果不设置主键，默认会找一个唯一的且不为NULL的列作为其隐式主键！如果设置了主键，则以主键构造一颗B+树，叶子节点存放数据行；

2.聚簇索引的叶子节点就是数据节点，而非聚簇索引的叶子节点仍然是索引节点，只不过有指向对应数据块的指针；

3.在InnoDB中，主键索引就是聚簇索引。MyISAM中主键索引为非聚簇索引，因为其数据文件与索引文件是分开的；

4.InnoDB中非聚簇索引与数据是分开的，但是保存在同一个文件中；

5.一个表只能拥有一个聚集索引，可以有多个非聚集索引；

6.聚簇索引中的每个叶子节点包含主键值、事务ID、回滚指针(rollback pointer用于事务和MVCC）和余下的列(如col2)；

主键索引就是一种聚簇索引，二级索引就是非聚簇索引。

一张表，除了聚簇索引以外的索引，都是二级索引（secondary index）；

二级索引，又称为：辅助索引；

一张表，二级索引可以有多个，没有上限；

二级索引，只存储部分数据；

在常用的查询列上建立二级索引；

不建议在大字段上建立二级索引；

=====================

=====================

# 索引下推

https://segmentfault.com/a/1190000039869289

index condition pushdown， ICP。
是MySQL 5.6 发布后，针对 扫描 二级索引的 一项优化。

总的来说 是通过把索引 过滤条件下推到 存储引擎，来减少 MySQL 存储引擎 访问基表的次数 以及 MySQL 服务层 访问 存储引擎的次数。

ICP 适用于 MyISAM 和 InnoDB。

MySQL ICP 涉及的知识点：

1.  MySQL 服务层：也就是server 层，用来解析 SQL 语法，语义，生成查询计划，接管从MySQL 存储引擎上推 的数据 进行 二次过滤 等。
    
2.  MySQL 存储引擎层：按照MySQL 服务层下发的请求，通过索引 或 全表扫描 等方式 把数据上传到 MySQL 服务层
    
3.  MySQL 索引扫描：根据指定索引过滤条件(如 where id=1)，遍历索引找到索引键 对应的主键值后 回表 过滤剩余过滤条件。
    
4.  MySQL 索引过滤：通过索引扫描 并且基于 索引 进行 二次条件过滤后 再回表。
    

ICP 就是把以上的 索引扫描 和 索引过滤 合并在一起处理， 过滤后的 记录数据 下推到 存储引擎 后的 一种 索引优化策略。 这样做的优点：

1.  减少回表的操作次数
2.  减少上传到 MySQL server层的 数据

ICP默认开启，可以通过 优化器开发参数 关闭 ICP：optimizer\_switch='index\_condition_pushdown=off' 或者是在 SQL 层面通过 HINT 来关闭。

不使用ICP 索引扫描的过程
MySQL 存储引擎层 只是把 满足索引键值 对应的 整行 表记录 一条条取出，并上传给 MySQL 服务层。
MySQL 服务层 对接收到的 数据，使用 SQL 语句后面的 where 条件过滤，直到处理完最后一行记录，再一起返回给 客户端。

假设SQL语句为：

```text
(localhost:mysqld.sock)|(ytt)>select * from t1 where r1 = 1 and r2 like '%dog%' and r4 = 5\G

*************************** 1. row ***************************
id: 28965
f1: 81
f2: 89
f3: 100
f4: 35
r1: 1
r2: 12844bda dog 11ea a051 08002753f58d
r3: 17
r4: 5
1 row in set (0.00 sec)
```

关闭ICP的处理流程大概如下图：

![1](../_resources/9ae5eea39a4d4f74a37363db7e08ec76.png)

使用ICP 扫描的过程：

MySQL 存储引擎层，先根据 过滤条件中包含的索引键 确定索引 的区间，再在这个区间 的记录上 使用 包含 索引键的其他过滤条件 进行过滤，之后规避掉不满足的 索引记录，只根据 满足条件的 索引记录 回表 取回数据上传到 MySQL 服务层。

MySQL服务层 对接受到的 数据，使用where 子句 中 不包含索引列的 过滤条件 做最后的过滤，然后 返回数据给客户端。

![](../_resources/67454bb5c0fb4681a7cecf811504b113.png)
虚线表示 规避掉 的记录。 开启ICP 后 明显减少了 上传到 MySQL 存储引擎层，MySQL服务层 的记录条数，节省了IO。

查看 语句是否使用了ICP，只需要对语句进行 Explain。在Extra 信息里可以看到 ICP相关信息。

下面是关闭，开启 ICP 的 explain 结果。

```text
(localhost:mysqld.sock)|(ytt)>explain select /*+ no_icp (t1)  */ * from t1 where r1 = 1 and r2 like '%dog%' and r4 = 5\G

*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ref
possible_keys: idx_r4,idx_u1
          key: idx_u1
      key_len: 5
          ref: const
         rows: 325
     filtered: 0.12
        Extra: Using where
1 row in set, 1 warning (0.00 sec)

(localhost:mysqld.sock)|(ytt)>explain select * from t1 where r1 = 1 and r2 like '%dog%' and r4 = 5\G  *************************** 1. row ***************************

           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ref
possible_keys: idx_r4,idx_u1
          key: idx_u1
      key_len: 5
          ref: const
         rows: 325
     filtered: 0.12
        Extra: Using index condition; Using where
1 row in set, 1 warning (0.00 sec)
```

不过这个信息过于简单，无法 从执行计划结果 判断 ICP的 优劣。

可以通过以下几种方法 来查看ICP 带来的 直观性能提升。

1.  show status like '%handler%'

show status 可以查看 存储引擎 的相关 指标监控结果。从下面结果可以看出：指标 handler\_read\_next (表示 MySQL 存储引擎按照 索引键顺序读取 下一行记录的请求数，也就是说 这个值 表示 按照索引值 来访问基表的 请求数)。 在没有ICP时，325， 即 对基表读取请求了325次。 开启ICP后，这个值仅有14次。

```text
(localhost:mysqld.sock)|(ytt)>flush status;
Query OK, 0 rows affected (0.01 sec)

(localhost:mysqld.sock)|(ytt)> select /*+ no_icp (t1) */ * from t1 where r1 = 1 and r2 like '%dog%' and r4 = 5\G

*************************** 1. row ***************************
id: 28965
f1: 81
f2: 89
f3: 100
f4: 35
r1: 1
r2: 12844bda dog 11ea a051 08002753f58d
r3: 17
r4: 5
1 row in set (0.00 sec)

(localhost:mysqld.sock)|(ytt)>show status like '%handler%';
+----------------------------+-------+

| Variable_name              | Value |

+----------------------------+-------+

| Handler_commit             | 1     |
| Handler_delete             | 0     |
| Handler_discover           | 0     |
| Handler_external_lock      | 2     |
| Handler_mrr_init           | 0     |
| Handler_prepare            | 0     |
| Handler_read_first         | 0     |
| Handler_read_key           | 1     |
| Handler_read_last          | 0     |
| Handler_read_next          | 325   |
| Handler_read_prev          | 0     |
| Handler_read_rnd           | 0     |
| Handler_read_rnd_next      | 0     |
| Handler_rollback           | 0     |
| Handler_savepoint          | 0     |
| Handler_savepoint_rollback | 0     |
| Handler_update             | 0     |
| Handler_write              | 0     |

+----------------------------+-------+
18 rows in set (0.00 sec)

(localhost:mysqld.sock)|(ytt)>flush status;
Query OK, 0 rows affected (0.01 sec)

(localhost:mysqld.sock)|(ytt)>select * from t1 where r1 = 1 and r2 like '%dog%' and r4 = 5\G

*************************** 1. row ***************************
id: 28965
f1: 81
f2: 89
f3: 100
f4: 35
r1: 1
r2: 12844bda dog 11ea a051 08002753f58d
r3: 17
r4: 5
1 row in set (0.00 sec)

(localhost:mysqld.sock)|(ytt)>show status like '%handler%';
+----------------------------+-------+

| Variable_name              | Value |

+----------------------------+-------+

| Handler_commit             | 1     |
| Handler_delete             | 0     |
| Handler_discover           | 0     |
| Handler_external_lock      | 2     |
| Handler_mrr_init           | 0     |
| Handler_prepare            | 0     |
| Handler_read_first         | 0     |
| Handler_read_key           | 1     |
| Handler_read_last          | 0     |
| Handler_read_next           | 14    |
| Handler_read_prev          | 0     |
| Handler_read_rnd           | 0     |
| Handler_read_rnd_next      | 0     |
| Handler_rollback           | 0     |
| Handler_savepoint          | 0     |
| Handler_savepoint_rollback | 0     |
| Handler_update             | 0     |
| Handler_write              | 0     |

+----------------------------+-------+
18 rows in set (0.00 sec)

(localhost:mysqld.sock)|(ytt)>
```

1.  开启 profiles
    查看profile 结果的总体时间，关闭 ICP 为：0.00101900，开启ICP为：0.00100325。时间上 ICP 占优势。

需要下推到底层存储的操作一般有很多限制，ICP限制如下：

1.  ICP 仅用于 需要访问基表 所有记录时使用，使用的访问方法为：range,ref,eq\_ref,ref\_or_null。 上面举的例子就是 ref 类型，ICP尤其是对 联合索引的 部分列 模糊查找非常有效
    
2.  ICP 同样适用于 分区表
    
3.  ICP 的目标是 减少 全行记录读取，从而减少I/O操作，仅用于 二级索引。主键索引 本身就是 表数据，不存在下推操作。
    
4.  ICP 不支持基于 虚拟列上建立的 索引，比如 函数索引。
    
5.  ICP 不支持 引用子查询的 条件。
    

=====================

=====================

# MySQL 加锁规则

https://juejin.cn/post/7121513578523263013

无脑三板斧：1\. 建立了索引加速查询、2. 关闭自动提交事务、3. 在需要确保原子性的数据库操作之间手动创建和提交事务。

仿佛实际开发 和 MySQL相关名词：读写锁、间隙锁、多版本并发控制、redo log、bin log、undo log毫不相干

当然导致数据库访问速度变慢的原因有很多：
sql语句编写不规范、
数据库服务器的性能差、
网络状况不佳等，
但是本文所侧重的点在于探究MySQL的锁机制，在其中发挥了什么作用。

MySQL 锁有哪几种

全局锁
MySQL 可以通过显示命令 对整个数据库实例 加全局读锁
flush tables with read lock
unlock tables
此时，整个数据库处于 只读状态，所有数据记录的更新，数据库/表 结构的改动 提交 都会被阻塞。这可以用于 全库的 数据备份。

表级锁
表锁
表锁可以通过下面 显示命令 实现 对 一个表 添加 读/写 锁
lock tables t read/write
unlock tables

如果线程A 为 t1 表 添加了 读锁， 为 t2 表添加了 写锁。 则其他线程 只能 读t1，写t1 被阻塞； 读/写 t2 都被阻塞。 线程A 在执行 unlock tables 之前，也只能 执行 读t1，读/写 t2 的操作。

元数据锁 (metadata lock)

MDL 锁 不需要 显式使用， 在访问一个表的时候 被自动加上，并且当事务完成提交时释放。当对一个表数据做 CRUD 时，自动加 MDL锁。 对表结构做出改动时，自动加MDL锁。

读锁之间不互斥，因此 多个线程可以同时访问 一张数据表。
读写锁之间，写锁之间 互斥(被读锁占用时，加写锁 的线程会被阻塞； 被写锁占用时，加 读锁/写锁 的线程 会被阻塞)

![1](../_resources/698ae26f00744ad6b3eaebe5251bc47d.jpg)

B因为 A事务还没有提交，所以 申请MDL写锁 被阻塞。 C申请MDL 读锁，但是被阻塞在 B申请 MDL写锁 之后，所以 在A 事务提交之前，表 完全丧失 读写能力。

行级锁

所谓的 读写锁 并不是单指 一个锁 叫 读锁/写锁，而是指 不同粒度的 锁 有 读锁 和 写锁 2种状态，允许的并发程序 也有所不同。

行级锁也是如此，行锁的存在 也使得 事务并发访问数据库的 性能进一步的提高，并依旧有 读写锁 之分。

但区别于 全局锁 和 表级锁， MySQL 行锁 是由 各个存储引擎 自己实现的，并不是所有的 存储引擎 都支持 行锁 (MyISAM 不支持)。

两阶段锁协议
InnoDB事务中，行锁在需要的时候添加，并直到 事务提交才释放 (锁的添加 和 释放 分为 2个阶段进行)

![1](../_resources/aa76d05c1c9f4a6a95ecd871a9daa583.jpg)

事务A（线程A）在提交之前，占有id=1这条行记录的写锁，事务B（线程B）修改同一行的操作将被阻塞。

死锁与检测
死锁原本是OS的概念，意思是，多个线程 都在等待 其他线程释放自己需要的资源，使得这些线程 陷入 无限的等待

![1](../_resources/b0dc21ebcc134474b52f052a04391d35.jpg)

这个例子中，A B 事务分别占有了 id=1, id=2 的 写锁，使得 2个线程 在试图 获取 其他线程 占有的 锁资源时 陷入 死锁。

InnoDB 默认开启了 死锁检测，每个新来的 被阻塞的线程，都会 主动判断 是不是 自己的加入 导致死锁 (检测逻辑就是 判断自己需要的 行资源 是否被其他 线程的事务 占有)，一旦检测到，则回滚 当前线程的 事务，确保 其他线程 可以得到 执行。

如果有多个线程 修改同一条记录，一旦并发度很高，则会消耗大量CPU 到死锁检测上，使得 数据库IO 性能下降。

控制热点记录的 并发访问量，是提示数据库 IO 的重要前提

多版本并发控制(MVCC)
select 查询是否占用了 行记录的读锁呢？ 不完全正确。

MySQL的 InnoDB 引擎 用于 控制事务隔离级别 的 多版本并发控制

简而言之，就是 每条行 记录值 的变化 是由 一个链式的结构 组织的， 存放在 undo log 文档中，undo log 在事务发生回滚时，用来回溯 事务 对 行记录的修改过程。

InnoDB 存储引擎 默认事务隔离级别是 可重复度，简单来说，就是在事务A启动期间，普通的select 无法访问到 其他事务 对表记录的 改动。

快照读
select * from t where id=1;

对于普通的查询操作，InnoDB引擎管理的 表的行记录 变更是链式组织的，那么 每条记录就相当于 一个个快照，所以 普通的select 被称为 快照读，会读取到 自己可见的 最近一个版本 (不一定是最新版本)，快照读并不加锁(也就是 没有获取 读锁)

当前读
select * from t where id=1 lock in share mode;
select * from t where id=1 for update;

2种不同的 当前读方式，当前读 可以读到 undo log 版本链上的 最新记录， 不同之处在于， 第一条SQL 获取了 id=1 这条 行记录的 读锁 (如果其他事务已经有id=1行的写锁时 将被阻塞)； 第二条select 查询虽然也是当前读，但是 获得了 id=1 这条记录的 写锁 (其他事务已经 有 读/写 锁 是将被阻塞)

间隙锁
间隙锁的出现解决了 幻读问题。

幻读概述

在InnoDB 引擎 的可重复度 隔离级别下，普通查询是 快照读，不会看到 其他并发 事务插入的 数据，因此 幻读 只在 当前读 情况下 才会出现。

幻读指 当前读场景下， 查询到 其他并发事务 新插入的行 ( 读到其他事务 对行记录的修改，并不属于幻读，因为当前读 就是读取到 行记录的 最新版本)

![1](../_resources/ab74dabbebf4482bb5d719d500b45bd1.jpg)

在没有间隙锁的情况下：

1.  事务A读取了 所有 c=1 的数据，并添加了 for update，希望锁住所有 c=1 的记录
    
2.  在RR隔离级别下，所有扫描到的行 都会加 行锁，因为 c 字段没有 索引，比较c=1 的操作就需要全表扫描，因此，事务A的 第一条sql 在当前读的情况下，为整张表的 3条 行记录都添加了 写锁。
    
3.  事务B 并发插入一条数据，并成功
    
4.  事务A 的第二个sql 依然 查询 c=1 的记录，会获得 B事务的记录，从语义上违反了 第一条sql的目的 (原本打算锁定 所有 c=1 的记录，但是 突然又冒出一条记录)
    

这里的核心问题在于：即使所有扫描到的 行记录都 加上了锁，但是 依然无法阻止 新记录的 插入。 要避免幻读， 就要将 记录之间的 间隙锁定 \-\- 间隙锁。

Gap Lock

间隙锁 在 可重复读隔离级别下 才有效，所以 本文描述 都是 基于 RR 级别 (InnoDB 存储引擎 默认隔离级别)。这里给出 间隙锁 配合 行锁 工作的 一些规则：

1.  所有的锁是添加在索引上的
2.  加间隙锁的基本单位是 next-key lock (前开后闭区间)
3.  查找过程中访问到的记录 和区间才会加锁
4.  索引上的 等值查询，给 唯一索引加锁的时候，next-key lock 退化为 行锁
5.  索引上的 等值查询，向右遍历时 且 最后一个值 不满足 等值条件时，next-key lock 退化为 间隙锁
6.  唯一索引上的 范围查询 会访问到 不满足 条件的第一个值 为止

=====================

=====================

# 唯一索引失效(产生重复数据)

https://juejin.cn/post/7127986629213421576

明明加了唯一索引，为什么还是产生了重复数据？

如果唯一索引的字段中，出现了null值，则唯一性约束不会生效。

创建唯一索引的字段，都不能允许为null，否则mysql的唯一性约束可能会失效。

删除记录分为 物理删除， 逻辑删除
物理删除： delete from t where id=1;

逻辑删除： update t set delete=1 where id=1;

逻辑删除需要在表中额外加一个删除状态字段，用于记录数据是否被删除。
在业务查询的时候，过滤掉 已经删除的 字段。

对于这种逻辑删除，是没有办法加唯一索引的。

假设之前为 product 表的 name 和model 加了 唯一索引，如果用户把某条记录删除了，delete 变成1。 后来 用户又添加了 一模一样的 商品。

由于有 唯一索引的存在，用户第二次添加 商品会失败，即使 该商品 已经被删除了，也无法再 添加。

。。。我感觉是，我会 把 直接 insert， catch到 SQLException ( 这个唯一索引失败 会报什么异常类？还是靠 异常的Message 来区分？应该有 特定的 异常类吧。) ，然后 选择update已有的数据。

有人会说：把 name,model,delete 同时做出唯一索引 不就可以了？
答：这样做 确实可以 解决 用户 逻辑删除了 某个商品， 然后又重新添加时 添加不了的问题。但是，第二次添加的 物品 就无法删除了。

如果真的想给 包含 逻辑删除的表，增加唯一索引，该怎么办？

删除状态+1
只要delete > 0 ，就都是删除。
这样的话，每次删除时， 都获取 那条相同记录的 最大删除状态，然后+1.

增加时间戳字段
导致逻辑删除表，不好加唯一索引 最根本的地方 在 逻辑删除那里。
为什么不增加一个字段，专门处理 逻辑删除功能呢？
答：可以增加 时间戳 字段。  把 name,model,delte,timstamp 4个字段同时做出 唯一索引。
insert数据时，timestamp字段默认1。
然后一旦有逻辑删除操作，则自动往该字段写入时间戳。

这样，即使是同一条记录，逻辑删除多次，每次生成的时间戳也不一样，保证数据的唯一性。
时间戳一般精确到 秒。  高并发，可以精确到 毫秒。  但是还是有可能重复的。

增加id字段
增加timestamp 基本可以解决问题，但是在 极限情况下，可能还是会产生 重复数据。
增加 主键 字段：delete_id

该方案的思路和 增加时间戳字段一致，即，在添加数据时 给delete\_id 设置默认值1，逻辑删除时，给 delete\_id 赋值 当前记录的主键 id。

。。算是 主键副本吧。

把 name,model,delete,delete_id， 4个字段同时做出 唯一索引。

重复历史数据如何增加唯一索引？
增加delete_id 字段。
先获取 相同记录的 最大 id

select max(id),name,category,unit,model from product group by name,category,unit,model;

然后将 delete_id 设置为1.
然后将 其他相同记录的 delete_id 字段，设置为 当前 主键。
这样就可以区分历史的 重复数据了。
当所有的delete\_id 都设置以后，就可以为 name,model,delete,delete\_id 4个字段增加唯一索引了。
。。就是 找出这个产品的 最大id，这个最大 id 是 有效产品， 其他的 就是 无效的 历史产品。设置为 它们自己的主键。

。。这种找 一个产品的 最大id，怎么感觉 是子查询呢， 上面的 也可以。 不知道哪种快： select id from product as a where not exist (select 1 from product as b where b.id > a.id)

给大字段 加 唯一索引。
目前mysql innodb存储引擎中索引允许的最大长度是3072 bytes，其中unqiue key最大长度是1000 bytes。

增加hash字段

我们可以增加一个 hash 字段， 取 大字段的 hash值， 生成 一个较短的 新值。该值 可以通过一些 hash 算法 生成，固定 16位 或 32位。

带来一个问题： hash冲突。
如果还有其他字段可以区分，并且 业务上允许这种重复数据，不写入数据库，该方案也是可行的。
。。如果其他字段可以区分，那么 直接在其他字段上 建立 索引就完事了。不需要hash
。。感觉有点 无意义。  主要是 场景 没有。 也不好说。
。。我记得， 索引可以 索引 大字段的 前 x个字符的？  。 前缀索引。

不添加唯一索引
如果实在不好加唯一索引，就不加。通过其他技术手段保证唯一性。
如果新增数据的入口比较少，比如只有job，或者数据导入，可以单线程顺序执行，这样就能保证表中的数据不重复。
如果新增数据的入口比较多，最终都发mq消息，在mq消费者中单线程处理。
。。？ ？ ？ 这个是 单线程执行，然后 insert前 select 下？ 还是什么？

redis分布式锁
如果字段太大，在mysql中不好加唯一索引，那么 为什么不使用 redis分布式锁 呢？

如果直接 为 name,model,delete,delete_id 加 redis分布式锁，显然没有意义，效率也不会高。

可以用 name,model,delete,delete_id， 4个字段 生成 一个 hash值， 然后 锁 这个hash。

新增数据 \- 加redis分布式锁 - 查询数据库 - 不存在就insert
\ 锁失败返回            \ 存在则返回

。。？？？究竟什么场景需要 唯一索引？ 为什么不索引 id ？ 不索引 业务主键 ？

批量插入数据
有些人 可能认为 既然有redis 分布式锁，就可以不用 唯一索引了。
那你是 没有遇到， 批量插入数据的场景。
假设，你有一个 list ，里面的数据 需要批量插入到数据库。
如果使用 redis 分布式锁，你需要这样：
for(Product product: list) {
   try {
        String hash = hash(product);
        rLock.lock(hash);
        //查询数据
        //插入数据
    } catch (InterruptedException e) {
       log.error(e);
    } finally {
        rLock.unlock();
    }
}

在循环中，要给每条数据加锁。
这样性能肯定不好。

。。？redis 加个锁 有什么性能消耗？ 最多 加一个 list， 把无法获得锁的 add进去，然后 swap。 外面套一个  while(!list.empty)

当然，有人会说，使用 redis 的 pipeline 批量操作不就可以了？
也就是一次性 给 500条，或者1000条数据上锁， 最后一次性 释放这些锁？
想想就不靠谱，这个锁得多大。
极容易造成 锁超时，比如 业务代码 没有执行完，锁过期了。

。。。想到一个， 先全部 select 出来，而不是 现搜。。哦，还要注意 list里不能重复，因为 全部select 的 并不包含 list 中 未insert的。

针对这种批量操作，如果此时 使用 mysql 的 唯一索引，直接 批量 insert 即可。 一条sql 搞定。
数据库会自动判断，是否存在 重复数据。

。。唯一索引 就这用吗？ 那我觉得 超大字段 hash 或 前缀索引 好 (hash更好，更分散，前缀的话，可能有些前缀 是固定开头的。。)， 碰撞的几率不会很大。  碰撞以后 就 select 下， 用到 索引的 select 很快的。

=====================

https://juejin.cn/post/6972799795937148959

# MySQL 亿级数据分页的优化

我们在查看前几页的时候，发现速度非常快，比如 limit 200,25，瞬间就出来了。但是越往后，速度就越慢，特别是百万条之后，卡到不行，那这个是什么原理呢。先看一下我们翻页翻到后面时，查询的sql是怎样的：

select * from t\_name where c\_name1='xxx' order by c_name2 limit 2000000,25;

这种查询的慢，其实是因为limit后面的偏移量太大导致的。比如像上面的 limit 2000000,25 ，这个等同于数据库要扫描出 2000025条数据，然后再丢弃前面的 20000000条数据，返回剩下25条数据给用户，这种取法明显不合理。

分页操作通常会使用limit加上偏移量的办法实现，同时再加上合适的order by子句。但这会出现一个常见问题：当偏移量非常大的时候，它会导致MySQL扫描大量不需要的行然后再抛弃掉。

。。这个不能 二分下吗。。不过 得确保 随机访问， 而且 有一个值 代表了 前面有多少条数据。 。 这就意味着 得用 连续数据，或者说 可以更新 下标直接计算出 数据的 地址。 而且 不能delete，不然 前面有多少条数据 就 不准了。 刷新是不可能刷新的。。

。。skip table ？ 就是保存了 后续的 1 2 4 8 16 32 64.. 的那个。 还得抽取数据，不然 31 * 21亿 也爆炸的。

。。skip list ?
。。还得保证 order by 的顺序 和  现有的顺序 是一致的。。。。。。还是 遍历吧。。
。。应该加索引 之后的 遍历 会快点， 普通的索引 有 横向 指针吗？
。。应该有，而且就是 因为 横向 指针， 得一个个 遍历过去，导致 慢。

。。不过 检测到 这种 疯狂分页遍历的， 可以 打个 标记，下一次 如果 只是 limit 后移了，那么 从上次 的 开始。  得锁表，不然 中间 有数据插入的话，就。。 或者无所谓。遍历的时候，差一点两点数据 谁知道呢。  但不能删除，不然 你把 锚点删除了，就只能重新开始了。。。 无所谓，重新开始 也比 每次从头开始快。

\[SQL\]
SELECT a.empno,a.empname,a.job,a.sal,b.depno,b.depname

from emp a left join dep b on a.depno = b.depno order by a.id desc limit 100,25;

受影响的行: 0
时间: 0.001s
\[SQL\]
SELECT a.empno,a.empname,a.job,a.sal,b.depno,b.depname

from emp a left join dep b on a.depno = b.depno order by a.id desc limit 4800000,25;

受影响的行: 0
时间: 12.275s

解决方案

1.  使用索引覆盖 \+ 子查询优化
    因为我们有主键id，并且在上面建立了 索引， 所以可以先在索引树中 找到 开始位置的 id值，然后再根据找到的 id 查询行数据。

。。。那遍历的时候 能不能 直接 用 id between 100 and 200 这种增量查询，而不是 limit。  不过between 返回的数据量 是不定的，  不过 都是遍历，无所谓吧。

\[SQL\]
SELECT a.empno,a.empname,a.job,a.sal,b.depno,b.depname
from emp a left join dep b on a.depno = b.depno
where a.id >= (select id from emp order by id limit 100,1)
order by a.id limit 25;
受影响的行: 0
时间: 0.106s

\[SQL\]
SELECT a.empno,a.empname,a.job,a.sal,b.depno,b.depname
from emp a left join dep b on a.depno = b.depno
where a.id >= (select id from emp order by id limit 4800000,1)
order by a.id limit 25;
受影响的行: 0
时间: 1.541s

。。 ？ 这里不还是 limit 4800000 了吗？ 还是得 遍历过去啊。 有什么用？
。。但是看这里 时间 确实 少了 很多很多。

1.  起始位置重定义
    记住上次查找结果的 主键位置，避免使用 offset

\[SQL\]
SELECT a.id,a.empno,a.empname,a.job,a.sal,b.depno,b.depname
from emp a left join dep b on a.depno = b.depno
where a.id > 100 order by a.id limit 25;
受影响的行: 0
时间: 0.001s

\[SQL\]
SELECT a.id,a.empno,a.empname,a.job,a.sal,b.depno,b.depname
from emp a left join dep b on a.depno = b.depno
where a.id > 4800000
order by a.id limit 25;
受影响的行: 0
时间: 0.000s

。。？？？ id 肯定不是 480000 的。  不过 无所谓，就是上次的 最大 id。

这个效率是最好的，无论怎么分页，耗时基本都是一致的，因为他执行完条件之后，都只扫描了25条数据。
但是有个问题，只适合一页一页的分页，这样才能记住前一个分页的最后Id。如果用户跳着分页就有问题了，比如刚刚刷完第25页，马上跳到35页，数据就会不对。
这种的适合场景是类似百度搜索或者腾讯新闻那种滚轮往下拉，不断拉取不断加载的情况。这种延迟加载会保证数据不会跳跃着获取。

1.  降级策略
    配置limit的偏移量和获取数一个最大值，超过这个最大值，就返回空数据。
    超过这个值你已经不是在分页了，而是在刷数据了，如果确认要找数据，应该输入合适条件来缩小范围，而不是一页一页分页。
    这个跟我同事的想法大致一样：request的时候 如果offset大于某个数值就先返回一个4xx的错误。

=====================

=====================
https://juejin.cn/post/7179239346967412773

# MYSQL中的14个神仙功能

1.  group_concat
    把group 分组后，name相同的数据 的code 拼接到一起，组成一个 字符串，用 逗号 分隔
    select name,group_concat(code) from `user` group by name;
    
2.  char_length
    获得字符长度
    

select * from brand where name like '%苏三%' order by char_length(name) asc limit 5;

1.  locate
    获得 关键字 在 字符串中的位置

select * from brand where name like '%苏三%' order by char_length(name) asc, locate('苏三',name) asc limit 5,5;

先按长度 升序，长度相同，则 按关键字 从左到右排序，越靠左越在前面。
还可以使用 instr 和 position 函数，功能和 locate 类似。

1.  replace
    update brand set name=REPLACE(name,'A','B') where id=1;

update brand set name=REPLACE(name,' ','') where name like ' %';
update brand set name=REPLACE(name,' ','') where name like '% ';
替换前后空格

这个函数 还可以 替换 json格式中 的数据内容。

1.  now
    select now() from brand limit 1;
    返回： 2022-12-15 17:02:45

如果要带 毫秒，可以使用 now(3)
select now(3) from brand limit 1;
返回：2022-12-12 11:11:11.678

1.  insert into .. select
    普通的插入
    INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`)
    VALUES (5, '108', '苏三', '2022-09-02 19:42:21');

上面主要是用于插入少量并且已经确定的数据。但是 如果有大量的数据 需要插入，特别是 需要插入的数据 来源于 另外一张 或 多张表的结果集中。 此时就能使用 insert into .. select

INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`)
select null,code,name,now(3) from `order` where code in ('004','005');

1.  insert into .. ignore
    场景： 在插入 1000 个品牌前， 需要先 根据 name，判断下 是否存在。如果存在，则不插入数据， 如果不存在，才需要插入数据。

如果直接这样插入数据：
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`)
VALUES (123, '108', '苏三', now(3));

肯定不行，因为 brand 表的 name 字段 创建了 唯一索引，同时 该表中 已经 有一条 name 等于 苏三 的数据了。 会报错： duplicate entry '苏三' for key 'brand.ux_name'。

需要在插入前 判断下。
很多人 通过在 sql 语句后面拼接 not exists 语句， 也能达到 防止出现重复数据的目的，比如
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`)
select null,'108', '苏三',now(3)
from dual where  not exists (select * from `brand` where name='苏三');

更简单的方法： insert into .. ignore
INSERT ignore INTO `brand`(`id`, `code`, `name`, `edit_date`)
VALUES (123, '108', '苏三', now(3));
这样改造后，如果brand表 中没有 name 为 苏三的 数据，则可以直接插入成功。

如果 brand 表中 已经存在 name 为 苏三的 数据，则 该sql 也能正常执行，不会报错。因为 它会 忽略异常，返回的 执行结果 影响行数为0， 不会重复插入数据

1.  select .. for update
    MySQL 自带悲观锁，是一种排它锁，根据锁的粒度 从大到小 分为是 表锁，间隙锁，行锁。

begin;
select * from `user` where id=1 for update;

//业务逻辑处理

update `user` set score=score-1 where id=1;
commit;

这样 在一个事务中 使用 for update 锁住一行记录，其他事务 就不能在 该事务提交之前，去更新那行的数据。

注意 for update 前的 id 条件，必须是 表的 主键 或 唯一索引，不然 行锁可能会 失效，有可能变成 表锁。

1.  on duplicate key update
    通常，在插入数据前，一般会 先查询一下，该数据是否存在。如果不存在，则插入数据。如果已经存在，则不插入数据，而是 直接返回结果。
    如果 有一定的 并发量，这种做法 就可能会产生 重复的数据。

防止重复数据的做法很多，比如，加唯一索引，加分布式锁 等。

但是这些方案，都没法 做到 让 第二次请求 也更新数据，它们一般 会判断 已经存在 就直接返回了。
此时 可以使用 on duplicate key update 语法
该语法会在 插入数据之前 判断，如果 主键 或 唯一索引不存在，则插入数据。如果 主键或 唯一index 存在， 则执行更新操作。
INSERT  INTO `brand`(`id`, `code`, `name`, `edit_date`)
VALUES (123, '108', '苏三', now(3))
on duplicate key update name='苏三',edit_date=now(3);

但是要注意，在 高并发的场景下 使用 on duplicate key update 语法，可能会存在 死锁的 问题，所以 要根据实际 情况酌情使用。

1.  show create table   show index
    有时，我们要 快速 查看 某张表的 字段情况，通常使用 desc 命令，比如
    desc `order`;

但是看不到 索引信息。
使用 show index 可以看到 索引信息
show index from `order`;

可以查出所有的 index， 但是 展现方式 有点奇怪，并不直观。
此时 需要 show create table
show create table `order`;

1.  create table .. select
    快速备份表
    通常情况下，分2步走
    创建一张临时表
    将数据插入到 临时表

创建临时表可以使用 命令
create table order_2022121819 like `order`;
创建成功后，就会生成 一张 名叫 order_2022121819 的表， 表结构 和 order 一模一样 的 新表， 数据为空。

insert into order_2022121819 select * from `order`;
将数据插入到 order_2022121819 中，实现 数据备份。

一个命令实现 上面的 2步
create table order_2022121820 select * from `order`;

1.  explain
    优化sql性能时，需要查看 index 执行情况， 可以使用 explain，查看mysql 的 执行计划，它会显示 索引的使用情况
    explain select * from `order` where code='002';

执行计划 各列 的含义

|     |     |
| --- | --- |
| id  | select 唯一标识 |
| select_type | select类型 |
| table | 表名  |
| partitions | 匹配的分区 |
| type | 连接类型 |
| possible_keys | 可能的索引选择 |
| key | 实际用到的index |
| key_len | 实际index长度 |
| ref | 与index比较的列 |
| rows | 预计要检查的行数 |
| filtered | 按表条件过滤的行百分比 |
| extra | 附加信息 |

sql没有走index， 排除 没有建index 之外，最大的可能就是 index失效
下面是 index 失效常见的原因
不满足最左前缀原则
范围索引列 没有放最后
使用了 select *
索引列上 有计算
索引列上 使用了函数
字符类型 没有加引号
用 is null, is not null 没有注意 字段是否允许为 空
like 查询 左边有 %
使用 or 关键字时 没有注意。

1.  show processlist
    数据库出现了问题，如 数据库连接过多，某条sql执行时间特别长。
    可以使用 show processlist 查看 当前线程 执行情况
    返回的 各列的含义

|     |     |
| --- | --- |
| id  | 线程id |
| User | 执行sql 的账号 |
| Host | 执行sql 的数据库的 ip 和端口 |
| db  | 数据库名称 |
| Command | 执行命令，包括 Daemon，Query，Sleep 等 |
| Time | 执行sql消耗的时间 |
| State | 执行状态 |
| info | 执行信息，里面可能包含sql信息 |

如果发现异常的 sql，可以直接 kill 掉，确保数据库 不会出现严重的问题。

1.  mysqldump

有时需要 导出 mysql 表的数据，可以使用 mysqldump，该工具 会将数据查出来，转成 insert 语句，写到某个文件，相当于 数据备份。

mysqldump命令的语法为： mysqldump -h主机名 -P端口 -u用户名 -p密码 参数1,参数2.... > 文件名称.sql

备份远程数据库中的数据库：
mysqldump -h 192.22.25.226 -u root -p123456 dbname > backup.sql

一些评论

now()，除了select外，其他写操作 要慎重使用， mysql主从复制的架构下，主从机 时间不一致 使用now()会导致 主从数据不一致。

mysql innodb 引擎，可以查询 information\_schema.innodb\_trx 确认长事务，并kill。

mysqldump 数据量很大时， 导出的 xxx.sql 文件占用空间很大，dump参数 支持 压缩导出(gzip)， 不要在 线上运行的数据库 使用 dump，会锁表， 在线迁移数据库 请使用 mysql tool

INSERT ignore INTO……select…… 插入不成功 也会影响 主键递增 的序号情况。

group_concat()有长度限制，超出会自动忽略后续字符。需要修改配置

如果能用到这些函数，肯定是我程序设计有问题

还有 regexp

replace有大坑，无法拿到真正受影响的行数，on duplicate key update 也有坑，阉割版的merge，不能指定冲突条件，超过一个唯一索引很容易出问题

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

删除会导致B+树的调整，索引的调整。  所以 fun(数据量,索引数量) > 多少 时， 状态字段 优于 直接delete？

=====================

将索引列由varchar(50)型改为bigint型后,数据提升了30倍。 究其原因就索引树上搜索时要进行大量的比较操作,而字符串的比较比整数的比较耗时的多。

=====================

char,varchar 的 前缀索引

=====================

对于Innodb来说，redundant或者compact类型的行格式，默认最大前缀索引长度是767；dynamic或者compressed行格式，默认的最大前缀索引长度是3072；

对于MyISAM来讲，最大前缀索引长度是1000。

=====================

1、InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；

2、 InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败；

3、InnoDB 是聚集索引，MyISAM 是非聚集索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

4、 InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；

5、InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；

=====================

=====================

https://baijiahao.baidu.com/s?id=1713340900738430106&wfr=spider&for=pc

# Mysql基础复习

很杂，还可以。

=====================

=====================

* * *

https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks-handling.html

# 15.7.5.3 How to Minimize and Handle Deadlocks

如何管理数据库操作 来最小化死锁。 应用中 需要的 error handling。

死锁 在 事务性数据库中 是一个 经典的问题， 但 通常没有危害，除非 频率很高，导致你无法 执行正常的 事务。
通常，你必须 在代码中 编写 由于死锁而导致的回滚 后的 处理逻辑。

InnoDB 使用 自动的 row-level locking。你可能 在 insert一行 或delete一行时 就 遇到死锁，这是因为这些操作 不是 原子性的， 它们自动 在 insert或delete 的 row 的 (可能多个) index 记录上 设置 锁。

你可以应对死锁，降低它们发生的概率，通过以下技术
使用 show engine innodb status 来 确定 最近的死锁的 原因。 这可以帮助你 调整应用 以避免死锁。

如果 频繁死锁，启用 innodb\_print\_all_deadlocks 来 收集 更多 信息。 当你完成debug 的时候，关闭这个 选项。

始终准备：由于死锁导致事务失败时 的重试。 死锁不是 危害性的。 just try again。

让 事务小 且 持续时间短 来 减小发生冲突的可能性。

在执行完 一组相关更改 后立即commit，来减少冲突。特别是，不要让 interactive mysql session 由于 未提交的事务 而 长时间 打开。

如果你使用 locking read (select .. for update/share)， 尝试使用 低级的 隔离级别，比如 read committed

当 在一个事务中修改多个表，或 一个表中不同的row集合，每次都以 一致的顺序 执行。那么 事务会 形成良好的队列，不会死锁。 例如，将 数据库操作 放到 function 中，或者 调用 stored routine, 而不是 多个 insert，uodpate，delete 的组合。

。。数据库的函数， 数据库的存储过程。

创建 精选的index，来使得 你的query scan 更少的index记录，设置更少的lock。 使用 explain select 来确定 MySQL server 认为 哪些index 最适合你的 query。

使用更少的locking，如果 可以接受 select 从old snapshot 返回 数据，那么不要 增加 for update,for share。 此时 使用 read committed 隔离级别 是good， 因为 同一个事务中的 每个 consistent read 从 它自己的 fresh snapshot 中读。

。。
普通读（也称一致性读，英文名：Consistent Read）。

这个就是指普通的SELECT语句，在末尾不加FOR UPDATE或者LOCK IN SHARE MODE的SELECT语句。普通读的执行方式是生成ReadView直接利用MVCC机制来进行读取，并不会对记录进行加锁。

。。

如果没有其他的办法有效，那么 使用 table-level lock 序列化 事务。正确的 对transactional table(比如InnoDB的表) 使用 lock tables 的方式是： 使用 set autocommit=0 (而不是 start transaction) 来启动 事务，然后 lock tables， 不要调用 unlock tables 直到 你显式 commit掉事务。 例如，你想要 对表t1写，对表t2读，你可以：

SET autocommit=0;
LOCK TABLES t1 WRITE, t2 READ, ...;
... do something with tables t1 and t2 here ...
COMMIT;
UNLOCK TABLES;
表级锁阻止 对表的并发update，避免死锁，代价是系统响应能力。

。。set autocommit=0指事务非自动提交,自此句执行以后,每个SQL语句或者语句块所在的事务都需要显示"commit"才能提交事务

另一个序列化 事务 的方式是 创建一个 辅助的 "semaphore" 表，只包含一行数据。每个 事务 在 访问其他表之前 都需要 更新那行。这样，所有的 事务 都以 序列化的顺序 发生。

注意，innodb 的 死锁侦测 也会起效，因为 序列化用的锁 是 row-level lock。对于 MySQL 表级锁 ，必须使用 超时 来解决死锁。

=====================

=====================

=====================

=====================

=====================

=====================

=====================

=====================

已使用 Microsoft OneNote 2016 创建。

=====================

https://blog.csdn.net/weixin_46594796/article/details/136257003

# buffer pool

## buffer pool 介绍

是 InnoDB 特有的内存结构。
mysql向OS 申请一块内存空间用于 buffer pool。

buffer pool 用于协调CPU速度和硬盘速度 之间的鸿沟。
能大幅提高 读写性能

对于查询操作，mysql会先搜索buffer pool，不存在 再去硬盘查询。


从OS角度，buffer pool在内存中的样貌：

![e8c9218a19f7ba6e111c7dc893a183dc.png](../_resources/e8c9218a19f7ba6e111c7dc893a183dc.png)


buffer pool 中有 数据页，里面保存了 硬盘中的数据。  
buffer pool默认 128MB。

innodb中，硬盘中 页 是 16kb，通过 页为基本单位，进行 磁盘和内存之间的交互。

可以申请多个 buffer pool，同时工作。

### 后台线程

buffer pool 中 缓存的 页 会 异步刷新到硬盘，保证数据一致性。

后台线程的主要作用就是 对缓冲池 中的 页 进行操作。

后台线程主要有以下几种
1. master thread
2. io thread
3. purge thread
4. page cleaner thread

master thread
主要用于 将buffer pool 中 缓存的数据 进行 刷盘，保证数据一致性，主要职责是： 脏页刷盘，插入缓冲合并，undo页回收

io thread
主要处理 AIO (async io) 请求回调。 innodb中存在大量的 异步io操作， io thread 极大影响了数据库性能

purge thread
事务提交后， undo页 就没有 存在的意义了，该线程主要职责就是 回收 无用的 undo页。  
上面提到 master thread 的职责包括 回收undo页， 但后续的 innodb 版本 会将部分 purge 操作 交给 purge thread，来减少 master thread的 工作压力。

page cleaner thread
也是分担 master thread 的工作，提升数据库性能。 分担了 脏页刷盘。


### redo log buffer

硬盘中 存在 redo log file，主要用于 故障恢复，保证 mysql 事务的 持久性， redo log buffer 就是用于存放 redo log 的，然后按照一定频率 刷盘到 redo log file中。


## buffer pool 组成

### 数据页

当查询的数据不存在于 缓冲池中时，会将 磁盘中的数据对应的页 加载到 buffer pool中，这就是 数据页。  
对数据页进行修改后，就变成了 脏页。 后续会把 脏页刷盘到 磁盘中。

以 页 为交互单位的 性能好很多。

### 索引页

buffer pool 不仅存放 数据页，还存放索引页。

如果查询命中了索引，我们该如何直到 索引的根节点 到底在磁盘的 哪个位置呢？ 
这时需要索引页来帮助我们，当mysql启动时，会将数据库中的 索引根节点 放入缓冲池的 索引页中，当我们的查询命中索引时，就不需要在整个磁盘中搜索 对应的 索引根节点了。

### undo页

记录事务回滚操作信息，常用于 事务回滚操作


### 插入缓冲

插入缓冲 只针对 非聚集，不唯一 索引页 增 删 改 操作

当我们对 非聚集，不唯一 的索引页进行 插入，修改操作时，不是直接操作索引页，而是先判断 当前索引页是否 在 buffer pool 中，  
如果在，直接操作 索引页即可，  
如果不在，就放入 insert buffer 对象中， 然后 以一定的频率进行 插入缓冲 和 辅助索引页 合并操作，大大提升 非聚集索引操作性能

为什么 聚集索引不需要 插入缓冲？ 因为 聚集索引 是按照主键顺序递增的，是 顺序插入，不需要 随机读取硬盘，性能很好。


### 锁空间

存储 锁结构，并发事务的链表 的一块内存区域，这里不多介绍。


### 数据字典

mysql启动时，会自动从 磁盘中将 系统表 相关信息加载到 buffer pool中，有了数据字典，这样当我们使用 show index, show tables 时就能 查到 表，索引 相关的信息。主要分为
1. sys_tables
2. sys_columns
3. sys_indexes
4. sys_fields


### 自适应hash索引

默认情况下， 索引页使用 B+ 树。

自适应hash索引，不需要人为干涉，由innodb自动生成。  
自适应hash索引，针对 热点索引页，不是 整张表，并且 生成的条件比较苛刻。  
当我们对 某个索引页连续的访问模式条件一样，如 where a=xxx 或 where a=xxx and b=yyy， 这2种 不能交替进行。   
当 以某个模式访问100次 或 n次 (n = 页中数据/16)  时 生成


## buffer pool 内存管理

### 控制块

innodb 从 OS中 为 buffer pool申请了一块 连续的内存， 内存被划分为 一块块 缓冲页 (来映射 磁盘中的 页)， innodb 为 每个 缓冲页 生成一个 控制块。

控制块中 记录了 数据页 所属的 表空间，页号，缓冲页地址，链表节点指针 等信息。


### free list

空闲页 被 free list 管理。 以方便快速查找

当硬盘中的 页 需要放入到 buffer pool时，就会从 free list 中查找是否有 空页，有就直接使用，没有就从LRU list 尾端获取

free list 是一个 双向链表，节点还保存了控制块的指针。头节点是 dummy节点，保存了 头，尾，节点个数

### flush list

脏页 通过 flush list来管理。


### LRU list

第一次插入插入在 从头开始计算的 5/8 处。

mysql有一个 配置 innodb_old_blocks_time 控制 等待多久才会添加到头，默认1000ms， 如果 第二次访问 距离第一次访问 超过这个时间间隔，那么会 把 缓存页 移动到 头 (而非 5/8处)










# 慢查询总结

@See [高性能MySQL](../Book/高性能MySQL.md)


## 开启慢查询日志

查看当前 慢日志的配置
`SHOW VARIABLES LIKE 'slow_query_log%';`

修改配置
```text
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
```

需要重启 mysql


分析慢日志
`mysqldumpslow /var/log/mysql/mysql-slow.log`


定期检查，删除日志


## mysql

### explain

[MySQL-Z](../扩展/MySQL-Z.md)

避免全表扫描，可以通过 索引 或 表分区

### 配置



- buffer-pool
- 划分大内存
- SSD


## 业务代码

- batch。 一次查询多条数据
- cache
  - mysql的buffer-pool
  - redis
  - ConcurrentHashMap
- 冗余。 避免join
- 1+n。 小表直接select，不走join
- select *。
- 代码实现sql函数。 数据库很难扩展，应用服务器很容易扩展

- 索引覆盖
- 索引下推
- join 小表在前
- 分页必带id

- 大事务

### 测试环境

- 基准测试
- 复现
  - 数据量
  - 并发压力
  - 冷热数据(测2次)
- 配置是否相同




# 表分区

内置，将一个表的数据分割成多个更小，更易于管理的部分。

## 创建分区表

在 `create table` 中使用 `partition by` 来指定分区类型和键
分区类型:
- 范围分区 range
- 列表分区 list
- 哈希分区 hash
- 键分区 key

假设 我们有sales表，要根据 销售日期来分区:
```sql
CREATE TABLE sales (
    sale_id INT AUTO_INCREMENT,
    product_id INT,
    sale_date DATE,
    amount DECIMAL(10, 2),
    PRIMARY KEY (sale_id, sale_date)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p0 VALUES LESS THAN (2020),
    PARTITION p1 VALUES LESS THAN (2021),
    PARTITION p2 VALUES LESS THAN (2022),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

上面的例子中，我们按照 sale_data 的年份进行分区，每个分区包含 特定年份的 所有销售记录


使用列表分区
假设 products表，要按照产品类型进行分区:
```sql
CREATE TABLE products (
    product_id INT,
    product_name VARCHAR(100),
    category_id INT,
    price DECIMAL(10, 2)
)
PARTITION BY LIST (category_id) (
    PARTITION pElectronics VALUES IN (1, 2, 3),
    PARTITION pClothing VALUES IN (4, 5),
    PARTITION pBooks VALUES IN (6)
);
```

我们根据 category_id 将产品分为 电子产品，服装，书籍， 3个分区

---

修改现有表，添加分区

```sql
ALTER TABLE sales PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p0 VALUES LESS THAN (2020),
    PARTITION p1 VALUES LESS THAN (2021),
    PARTITION p2 VALUES LESS THAN (2022),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```


## 查询

在所有分区中查询 的sql 和 普通sql 一样
```sql
SELECT * FROM your_partitioned_table WHERE partition_column = 'value';
```


---

指定特定分区

老版本，不支持直接指定 分区。 但是可以通过 过滤条件来限制查询范围
。。不清楚下面的 'partition_name' 是指 分区的名字? 但是这不是指定了吗。。
。。看这条sql 的感觉是: 分区后，会增加一列? 所以 不是增加了表? 没有增加列; 没有增加表，逻辑上 还是单个表，但是 mysql内部是分开存储的。

```sql
SELECT * FROM your_partitioned_table WHERE partition_column = 'value' AND partition_name = 'partition_name';
```

mysql 8.0 及之后:
```sql
SELECT * FROM your_partitioned_table PARTITION (partition_name) WHERE partition_column = 'value';
```

