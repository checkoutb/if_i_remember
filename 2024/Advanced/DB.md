DB
2021年9月1日
9:11

=====================================
数据库范式

范式：normal form  简称  FN

1NF
在关系模型中，所有的域都应该是原子性的，即数据库表的  每一列  都是不可分割的原子数据项，而不能是  集合，数组，等非原子数据项。
即实体中某个属性  有多个值时，必须拆分为  不同的  属性。
在  符合  1NF  的表中，每个列  都只能是  实体的一个属性  或  一个属性的一部分。
简而言之，第一范式  就是  无重复的域。

2NF

在1NF  的基础上，  要求数据库表  中的每个实例  或记录  必须可以被  唯一地区分。选取一个  能区分  每个实体的  属性或  属性组，作为  实体的  唯一标识。

2NF  要求  实体的属性  完全依赖于  主关键字。  所谓完全依赖  是指  不能存在  仅依赖  主关键字  一部分的  属性，如果存在，那么这个属性  和  主关键字  的这一部分  应该  分离出来  形成一个新的实体，  新实体  和  原实体  之间是  1对多的  关系。为了实现  区分  通常要为  表  加上一个列，以存储  各个实例的  唯一标识。

简而言之，2NF  就是在  1NF  的基础上   属性完全依赖于  主键。

3NF
2NF  的基础上，任何非  主属性  不依赖于  其他  非主属性  ，在  2NF的  基础上  消除  传递依赖。
简而言之，3NF要求  一个  关系中  不包含  已在  其他关系  已包含的  非主关键字  信息。

例如，有一个  部门信息表，每个部门有  编号，名称，简介  等。那么  在员工信息表  中  列出  部门编号后，就不能  再将  部门名称  部门简介  的信息  加入到  员工表中。

如果不存在部门信息表，则根据  3NF，就应该  构建一个  部门信息表，否则有  大量冗余数据。
简而言之，3NF  就是  属性不依赖于  其他非主属性，也就是  在  2NF的基础上，任何非主属性  不得  传递依赖于  主属性。

BCNF
Boyce-Codd Normal Form
巴斯-科德范式
3NF的基础上，任何主属性  不能对  主键子集  依赖  (在  3NF  的基础上，消除  主属性  对  主键  子集的  依赖)
通常情况下，BCNF  被认为  没有  新的设计规范加入，只是  对  2NF 3NF  的增强。因此不被认为是  4NF。

4NF

5NF

。。。没了。。。

------------------------
1NF
强调  属性的  原子性约束，要求  属性具有原子性，不可再分解。

人员表(姓名，年龄，地址)，因为  地址可以细分为  国家，省，市等，所以  没有达到  1NF。

2NF

强调  记录的  唯一性约束，数据表  必须有一个  主键，并且  没有在  主键中的  属性  必须完全  依赖于  主键，且  不能依赖  主键的  一部分。

版本表(  版本编码，版本名称，产品编码，产品名称)，主键是  (版本编码，产品编码)，  不符合  2NF，因为  产品名称  只依赖于  产品编码。  存在  部分依赖，  所以  为了满足  2NF，可以改为  2个表：  版本表(版本编码，版本名称，产品编码)，产品表(产品编码，产品名称)

3NF
强调  数据属性的  冗余性  的约束，也就是  非  主键列  必须直接依赖于  主键，也就是  消除了  非主属性  对  主键  的  传递依赖。

订单表(订单编码，顾客编码，顾客名称)，主键是  (订单编码)，  这个产品中，顾客编码，顾客名称  都是  完全依赖于主键的，  所以符合  2NF，  但是  顾客名称  依赖于  顾客编码，从而  间接依赖于  主键，所以  不满足  3NF。   需要拆分为  2个表，订单表，顾客表。

3NF  肯定  满足  2NF。产生冗余  和  异常的  2个重要原因是   部分依赖  和  传递依赖。  3NF  中不存在  非主属性  对  主键的  部分依赖  和  传递依赖，性能较好。

1NF，2NF  一般  不适合作为  数据库模式，通常  需要转换为  3NF  或更高，  这种变换过程  被称为  关系模式规范化处理

BCNF

属于  修正的  第三范式，是防止  主键的  某一列  会  依赖于  主键的  其他列。  即在  3NF的  基础上，消除了  主属性  对  主键  的  部分依赖  和  传递依赖。

特性：
1、所有主属性对每一个码都是完全函数依赖
2、所有主属性对每一个不包含它的码，也是完全函数依赖
3、没有任何属性完全函数依赖与非码的任何一组属性

举例：库存表（仓库名，管理员名，商品名，数量），主键为（仓库名，管理员名，商品名），这是满足前面三个范式的，但是仓库名和管理员名之间存在依赖关系，因此删除某一个仓库，会导致管理员也被删除，这样就不满足BCNF。

4NF
非主属性  不应该有多值。  如果有多值  就违反了  4NF。
4NF  是限制  关系模式  的  属性间  不允许有  非平凡  且  非函数依赖  的  多值依赖。

举例：用户联系方式表（用户id，固定电话，移动电话），其中用户id是主键，这个满足了BCNF,但是一个用户有可能会有多个固定电话或者多个移动电话，那么这种设计就不合理，应该改为（用户id，联系方式类型，电话号码）。

说明：如果只考虑函数依赖，关系模式规范化程度最高的范式是BCNF；如果考虑多值依赖则是4NF。

5NF
最终范式，消除  4NF  中的  连接依赖。
第五范式需要满足以下要求：
1、必须满足第四范式
2、表必须可以分解为较小的表，除非那些表在逻辑上拥有与原始表相同的主键。

实际应用中  一般不考虑

------------------------------

https://zhuanlan.zhihu.com/p/375068363

数据库范式内容挺多的。

=====================================

=====================================

=====================================

垂直分表：可以把一个宽表的字段按访问频次、是否是大字段的原则拆分为多个表，这样既能使业务清晰，还能提升部分性能。拆分后，尽量从业务角度避免联查，否则性能方面将得不偿失。

垂直分库：可以把多个表按业务耦合松紧归类，分别存放在不同的库，这些库可以分布在不同服务器，从而使访问压力被多服务器负载，大大提升性能，同时能提高整体架构的业务清晰度，不同的业务库可根据自身情况定制优化方案。但是它需要解决跨库带来的所有复杂问题。

水平分库：可以把一个表的数据(按数据行)分到多个不同的库，每个库只有这个表的部分数据，这些库可以分布在不同服务器，从而使访问压力被多服务器负载，大大提升性能。它不仅需要解决跨库带来的所有复杂问题，还要解决数据路由的问题(数据路由问题后边介绍)。

水平分表：可以把一个表的数据(按数据行)分到多个同一个数据库的多张表中，每个表只有这个表的部分数据，这样做能小幅提升性能，它仅仅作为水平分库的一个补充优化。

一般来说，在系统设计阶段就应该根据业务耦合松紧来确定垂直分库，垂直分表方案，在数据量及访问压力不是特别大的情况，首先考虑缓存、读写分离、索引技术等方案。若数据量极大，且持续增长，再考虑水平分库水平分表方案。

=====================================

读写分离
主库写入，从库读取

多机房还是  一个主库。  其他机房的写入是跨机房写入到  主库所在的机房。每个机房都有从库。

使用读写分离的原因：
读写量很大，为了提升数据库读写性能，将读写进行分离
多机房下如果写少读多，同时基于数据一致性考虑，只有一个主库存入所有的数据写入，本地再做从库提供读取，减少多机房间直接读取带来的时间延迟。

读写分离实现方案
如果要实现读写分离，我们首先要知道从业务层到数据库中间会经过哪几层，了解了这几层，我们的思路就是分别从这些层去实现读写分离，这样就构成了我们不同的实现方案。

在开发者的角度来看，系统分为  service-dao-jdbc-datasource-DB  层。

dao层
如果在这一层做读写分离，很容易想到的方案就是初始化2个ORM，一个读，一个写，根据需要调用对应的ORM。
简单的例子：
一个Sample数据表，包含id，名称，状态，创建时间。
主要操作：新增，获取，更新。
@Repository
public  class DaoImpl implements Dao {
@Autowired
private Jdbc readJdbc;
@Autowired
private Jdbc writeJdbc;
@Override
public boolean add(String appId, String name) {
StringBuilder sql = new StringBuilder();
sql.append(" insert into sample(app_id, name, status, create_time)");
sql.append(" values (?,?,?,?)");

StatementParameter param = new StatementParameter();
param.setString(appId);
param.setString(name);
param.setBool(true);
param.setDate(new Date());

return writeJdbc.insertForBoolean(sql.toString, param);
}
@Override
public Sample get(long appId) {            //  上面是string，这里变成了long…..
StringBuilder sql = new StringBuilder();
sql.append(" select * from sample where app_id = ?");

StatementParameter param = new StatementParameter();
param.setLong(appId);

return readJdbc.query(sql.toString(), Sample.class, param);
}
@Override
public Sample updateAndGet(String appId, boolean status) {
StringBuilder writeSql = new StringBuilder();
sql.append(" update sample set status = ? where app_id = ?");

StatementParameter writeParam = new StatementParameter();
writeParam.setBool(status);
writeParam.setString(appId);

writeJdbc.updateForBoolean(writeSql.toString(), readParam);

StringBuilder readSql = new StringBuilder();
readSql.append(" select * from sample where app_id = ?");

StatementParameter readParam = new StatementParameter();
readParam.setLong(appId);

//  依然使用write库，防止  从库还没有  被同步。
return writeJdbc.query(readSql.toString(), Sample.class, readParam);
}
}

优点
易于实现

缺点
侵入业务，每个数据操作需要考虑读写分离
对于读写窗口一致性要求操作者自行实现，考虑不周会导致数据读取失败。

JDBC层
由于DAO层读写分离  对业务侵入很大，读写分离需要业务方自行实现。

需要将JDBC层的  接口函数重写，对业务暴露一个  JDBCProxy，它通过读写决策器进行选择  此时是  使用读还是  写。jdbcWriter,jdbcReader  都是对jdbc接口的一个实现：

public class JdbcProxyImpl implements Jdbc {

private final static Logger logger = LoggerFactory.getLogger(JdbcProxyImpl.class);

private JdbcReaderImpl jdbcReader;
private JdbcWriterImpl jdbcWriter;

public void setJdbcReader(xxx) {xxx}
public void setJdbcWriter(xxxx) {xxxx}

@Override
public int update(String sql) {
//  更新时首先需要标记为  写入，再调用jdbc来实现更新
ReadWriteDataSourceDecision.markWrite();
return jdbcWriter.update(sql);
}

@Override
public <T> T query(String sql, Class<T> elementType) {
if (ReadWriteDataSourceDecision.isChoiceWrite()) {
return jdbcWriter.query(sql, elementType);
}
return jdbcReader.query(sql, elementType);
}
public boolean commit() {
return jdbcWriter.commit();
}
}

对于查询操作，会从ReadWriteDataSourceDecision中进行判断：
public  class ReadWriteDataSourceDecision {
public enum DataSourceType{
write, read;
}

private static final ThreadLocal<DataSourceType> holder = new ThreadLocal<>();

public static void markWrite() {
holder.set(DataSourceType.write);
}

public static void markRead() {
holder.set(DataSourceType.read);
}

public static void reset() {
holder.set(null)
}

public static boolean isChoiceNone() {
return null == holder.get();
}

public static boolean isChoiceWrite() {
return DataSourceType.write == holder.get();
}

public static boolean isChoiceRead() {
return DataSourceType.read == holder.get();
}
}

使用ThreadLocal  来保存当前线程的  读写属性，可以识别到当前线程是否有  写入操作，如果有写入，就使用  jdbcWriter  来进行读写，  保证  读写窗口的一致性。

DataSource层
jdbc层很好地解决了侵入性以及窗口一致性问题，但是我们在配置的时候仍然需要为proxy配置2个jdbc，相应的参数需要完整设置一遍，还是比较麻烦。。。

有没有只需要告诉它读写分离的  jdbc url，剩下的一个组件内部全部搞定呢？另外如果主从复制延迟较大，如果当前线程是  纯读取，也要看是否  时延超过了容忍阈值，如果超过了则仍然需要强制从主库读取，这就是DataSource层的方案

读写分离：  通过实现DataSource的connection以及statement，当jdbc请求执行sql时，会首先获取connection，通过解析sql判断是查询还是更新  来选择  连接池的读写连接类型，同时需要结合  主从复制检测的结果进行综合判断  来实现  读写连接分离。而读写的连接都是从读写的DataSource中获取。

读写窗口一致性：  通过重写DataSource的connection，如果当前连接已经存在写请求就强制采用写连接。

主从复制  时延职能切换：  通过启动单线程检测master和slave数据是否存在时延，来决策系统主从是否存在时延，如果存在时延就强制系统本次执行主库查询。

connection中几个重要方法：

public  PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) throws SQLException {

//  通过sql  解析是否  是读请求，并且  时延  没有超过预定阈值，则请求读连接，否则取写连接
if  (SqlUtil.isReadRequest(sql) && !dataSource.getMdsm().isDelay()) {

return getReadConnection().prepareStatement(sql, resultSetType, resultSetConcurrency);

} else {

return getWriteConnection().prepareStatement(sql, resultSetType, resultSetConcurrency);

}
}

Connection getReadConnection() throws SQLException {
if (writeConn  != null) {
//  如果是读数据，并且已经有写连接，那么强制返回这个写连接
return writeConn;
}
if (readConn != null) {
return readConn;
} else {
readConn = dataSource.getReadConnection(userName, password);
}
return readConn;
}

=====================================

=====================================

=====================================

聚簇索引：聚簇顾名思义，聚集在一起，即索引和数据是存放同一个文件中。其叶子节点中存放的就是整张表的行记录数据，也将聚集索引的叶子节点称为数据页。InnoDB引擎使用的是非聚簇索引。

非聚簇索引：索引文件和数据文件是分开的。MyISAM引擎默认使用的是非聚簇索引。

数据库索引类型有普通索引、组合索引、唯一索引、全文索引、主键索引

1、普通索引，基本的索引，没有任何限制，用于加速查询，数据可以重复

2、组合索引，指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用

3、全文索引，用来查找文本中的关键字

4、唯一索引，索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

5、主键索引，特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引。也就是在唯一索引的基础上相应的列必须为主键

4. 单列索引、多列索引
多个单列索引与单个多列索引的查询效果不同，因为执行查询时，MySQL只能使用一个索引，会从多个索引中选择一个限制最为严格的索引。

5. 组合索引（最左前缀）

平时用的SQL查询语句一般都有比较多的限制条件，所以为了进一步榨取MySQL的效率，就要考虑建立组合索引。例如上表中针对title和time建立一个组合索引：ALTER TABLE article ADD INDEX index_titme_time (title(50),time(10))。建立这样的组合索引，其实是相当于分别建立了下面两组组合索引：

–title,time
–title

为什么没有time这样的组合索引呢？这是因为MySQL组合索引“最左前缀”的结果。简单的理解就是只从最左面的开始组合。并不是只要包含这两列的查询都会用到该组合索引.

只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。

对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的列，如果在前10个或20个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。

MySQL查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。

select * from users where YEAR(adddate)<2007，将在每个行上进行运算，这将导致索引失效而进行全表扫描，因此我们可以改成：select * from users where adddate<’2007-01-01′

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

=====================================

已使用 Microsoft OneNote 2016 创建。