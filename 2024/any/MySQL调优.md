
[[toc]]

2023-07-07 16:00


视频08:30开始

# 这个视频不完整，大纲不错，但是太模糊了。好像没有办法抄下来。


==官网很重要！==

# 性能监控 


## show profile(s) !!!deprecated

官网说了：The SHOW PROFILE and SHOW PROFILES statements are deprecated; expect it to be removed in a future MySQL release。
Use the Performance Schema instead; see Section 27.19.1, “Query Profiling Using Performance Schema”.  
https://dev.mysql.com/doc/refman/8.0/en/performance-schema-query-profiling.html



`show profiles;`
如果为空，应该是没有开启监控

### 开启监控
默认不开启。
`set profiling=1;`

MySQL架构分为三层：
- 客户端
- server: 
  - 连接器(权限认证)，
  - 分析器(表名，字段名是否存在，是否符合sql，即语法语义，然后转化为AST (abstract syntax tree))，
  - 优化器(2个角度: CBO(cost-based optimization,基于成本的优化),RBO(rule-based op, 基于规则的优化), CBO用的更多)， 优化完成后，生成执行计划，即explain的内容。
  - 执行器
- storage engine


`show profile;`
会输出各个阶段的时间
包括：启动，权限校验，打开表，初始化，system lock，优化，统计，准备，执行，发送数据，结束，查询结束，关闭表，释放item，清理。


。。。
SHOW PROFILES 显示最近发送给mysql server 的语句列表，列表大小由 profiling_history_size 变量控制，默认为15条，最大为100条，设置为0则实际关闭该选项。

SHOW PROFILE和SHOW PROFILES语句本身不会被统计，但是非法的或错误的语句会被统计。

SHOW PROFILE显示单条语句的详细信息。如果没有FOR QUERY n语句，输出会包含最近执行过的多条语句；包括FOR QUERY n语句的话，SHOW PROFILE会只显示第n条语句的信息；n的值对应于Query_ID，改值可通过SHOW PROFILES查看。


`SELECT @@profiling;`
`SHOW PROFILES;`
`SHOW PROFILE;`
`SHOW PROFILE FOR QUERY 1;`

下面的语句是等价的
`SHOW PROFILE FOR QUERY 2;`

`SELECT STATE, FORMAT(DURATION, 6) AS DURATION FROM INFORMATION_SCHEMA.PROFILING WHERE QUERY_ID = 2 ORDER BY SEQ;`
。。。

还可以展示：
- IO次数
- 上下文切换次数
- CPU时间
- IPC时间
- 内存
- 等

`show profile cpu for query 1;`

`show profile all for query 1;`


## performance_schema

`show databases;`
可以看到 performance_schema

`use performance_schema;`

`show tables;`


对于开发者，没必要。
DBA需要。


The Performance Schema is a tool to help a DBA do performance tuning by taking real measurements instead of “wild guesses.” 
https://dev.mysql.com/doc/refman/8.0/en/performance-schema-examples.html



## 查看连接


`show processlist;`

可以查看连接的状态。
连接需要占用fd

关闭不需要的连接。。。不过没说怎么关。。

`ulimit -a`
可以看到 open files。就是 每个进程最多打开 多少个 fd

1g内存可以打开 10万个 fd。

`show variables like %max_connection%;`





# schema和数据类型优化

## 数据类型优化
更小的数据类型更好



# 执行计划




# 通过索引进行优化


# 查询优化


# 分区表


# 服务器参数设置


# MySQL集群



