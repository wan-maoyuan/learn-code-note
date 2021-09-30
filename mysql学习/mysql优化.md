## MySQL优化

### 数据库设计

- 表字段避免null值出现，null值很难查询优化且占用额外的索引空间，推荐默认数字0代替null。

- 尽量使用INT而非BIGINT，如果非负则加上UNSIGNED（这样数值容量会扩大一倍），当然能使用TINYINT、SMALLINT、MEDIUM_INT更好。

- 使用枚举或整数代替字符串类型。

- 尽量使用TIMESTAMP而非DATETIME。

- 单表不要有太多字段，建议在20以内。

- 用整型来存IP。


### 索引设计

- 索引并不是越多越好，要根据查询有针对性的创建，考虑在WHERE和ORDER 

- BY命令上涉及的列建立索引，可根据EXPLAIN来查看是否用了索引还是全表扫描。

- 应尽量避免在WHERE子句中对字段进行NULL值判断，否则将导致引擎放弃使用索引而进行全表扫描。

- 值分布很稀少的字段不适合建索引，例如"性别"这种只有两三个值的字段。

- 字符字段只建前缀索引。

- 字符字段最好不要做主键。

- 不用外键，由程序保证约束。

- 尽量不用UNIQUE，由程序保证约束。

- 使用多列索引时注意顺序和查询条件保持一致，同时删除不必要的单列索引。

- 使用可存下数据的最小的数据类型，整型 < date,time < char,varchar < blob*

- 使用简单的数据类型，整型比字符处理开销更小，因为字符串的比较更复杂。如，int类

- 型存储时间类型，bigint类型转ip函数。

- 使用合理的字段属性长度，固定长度的表会更快。使用enum、char而不是varchar。

- 尽可能使用not null定义字段。

- 尽量少用text，非用不可最好分表。

- 查询频繁的列，在where，group by，order by，on从句中出现的列。

- where条件中<，<=，=，>，>=，between，in，以及like 

- 字符串+通配符（%）出现的列。

- 长度小的列，索引字段越小越好，因为数据库的存储单位是页，一页中能存下的数据越多越好。

- 离散度大（不同的值多）的列，放在联合索引前面。查看离散度，通过统计不同的列值来实现，count越大，离散程度越高。


### SQL编写

- 使用limit对查询结果的记录进行限定。

- 避免select *，将需要查找的字段列出来。

- 使用连接（join）来代替子查询。

- 拆分大的delete或insert语句。

- 可通过开启慢查询日志来找出较慢的SQL。

- 不做列运算：SELECT id WHERE age + 1 = 10，任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移至等号右边。

- SQL语句尽可能简单：一条SQL只能在一个cpu运算；大语句拆小语句，减少锁时间；一条大SQL可以堵死整个库。

- OR改写成IN：OR的效率是n级别，IN的效率是log(n)级别，in的个数建议控制在200以内。

- 不用函数和触发器，在应用程序实现。

- 避免%xxx式查询。

- 少用JOIN。

- 使用同类型进行比较，比如用'123'和'123'比，123和123比。

- 尽量避免在WHERE子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

- 对于连续数值，使用BETWEEN不用IN：SELECT id FROM t WHERE num BETWEEN 1 AND 5。

- 列表数据不要拿全表，要使用LIMIT来分页，每页数量也不要太大。


### 分区

- 可以让单表存储更多的数据。

- 分区表的数据更容易维护，可以通过清楚整个分区批量删除大量数据，也可以增加新的分区来支持新插入的数据。另外，还可以对一个独立分区进行优化、检查、修复等操作。

- 部分查询能够从查询条件确定只落在少数分区上，速度会很快。

- 分区表的数据还可以分布在不同的物理设备上，从而搞笑利用多个硬件设备。

- 可以使用分区表赖避免某些特殊瓶颈，例如InnoDB单个索引的互斥访问、ext3文件系统的inode锁竞争。

- 可以备份和恢复单个分区。

- 一个表最多只能有1024个分区。

- 如果分区字段中有主键或者唯一索引的列，那么所有主键列和唯一索引列都必须包含进来。

- 分区表无法使用外键约束。

- NULL值会使分区过滤无效。

- 所有分区必须使用相同的存储引擎。


### 分表

- 分表就是把一张大表，按照如上过程都优化了，还是查询卡死，那就把这个表分成多张表，把一次查询分成多次查询，然后把结果组合返回给用户。

- 分表分为垂直拆分和水平拆分，通常以某个字段做拆分项。比如以id字段拆分为100张表：表名为 tableName_id%100。

- 但：分表需要修改源程序代码，会给开发带来大量工作，极大的增加了开发成本，故：只适合在开发初期就考虑到了大量数据存在，做好了分表处理，不适合应用上线了再做修改，成本太高！！！而且选择这个方案，都不如选择我提供的第二第三个方案的成本低！故不建议采用。


### 分库

- 把一个数据库分成多个，建议做个读写分离就行了，真正的做分库也会带来大量的开发成本，得不偿失！不推荐使用。


### 升级数据库

- 开源数据库会带来大量的运维成本且其工业品质和MySQL尚有差距，有很多坑要踩，如果你公司要求必须自建数据库，那么选择该类型产品。如tiDB pingcap/tidb，Cubrid Open Source Database With Enterprise Features。

- 阿里云POLARDB，POLARDB 是阿里云自研的下一代关系型分布式云原生数据库，100%兼容MySQL，存储容量最高可达 100T，性能最高提升至 MySQL 的 6 倍。POLARDB 既融合了商业数据库稳定、可靠、高性能的特征，又具有开源数据库简单、可扩展、持续迭代的优势，而成本只需商用数据库的 1/10。

- 阿里云OcenanBase，淘宝使用的，扛得住双十一，性能卓著，但是在公测中，我无法尝试，但值得期待。

- 阿里云HybridDB for MySQL (原PetaData)，云数据库HybridDB for MySQL （原名PetaData）是同时支持海量数据在线事务（OLTP）和在线分析（OLAP）的HTAP（Hybrid Transaction/Analytical Processing）关系型数据库。

- 腾讯云DCDB，DCDB又名TDSQL，一种兼容MySQL协议和语法，支持自动水平拆分的高性能分布式数据库——即业务显示为完整的逻辑表，数据却均匀的拆分到多个分片中；每个分片默认采用主备架构，提供灾备、恢复、监控、不停机扩容等全套解决方案，适用于TB或PB级的海量数据场景。


### 换大数据引擎

- hadoop家族。hbase/hive怼上就是了。但是有很高的运维成本，一般公司是玩不起的，没十万投入是不会有很好的产出的！

- 我选择了阿里云的MaxCompute配合DataWorks，使用超级舒服，按量付费，成本极低。

- MaxCompute可以理解为开源的Hive，提供SQL/mapreduce/ai算法/python脚本/shell脚本等方式操作数据，数据以表格的形式展现，以分布式方式存储，采用定时任务和批处理的方式处理数据。DataWorks提供了一种工作流的方式管理你的数据处理任务和调度监控。
- 
- 当然你也可以选择阿里云hbase等其他产品，我这里主要是离线处理，故选择MaxCompute，基本都是图形界面操作，大概写了300行SQL，费用不超过100块钱就解决了数据处理问题。

### EXPLAIN

> 做MySQL优化，我们要善用EXPLAIN查看SQL执行计划。

- type列，连接类型。一个好的SQL语句至少要达到range级别。杜绝出现all级别。
- key列，使用到的索引名。如果没有选择索引，值是NULL。可以采取强制索引方式。
- key_len列，索引长度。
- rows列，扫描行数。该值是个预估值。
- extra列，详细说明。注意，常见的不太友好的值，如下：Using filesort，Using temporary。

### SQL语句中IN包含的值不应过多

> MySQL对于IN做了相应的优化，即将IN中的常量全部存储在一个数组里面，而且这个数组是排好序的。但是如果数值较多，产生的消耗也是比较大的。再例如：select id from t where num in(1,2,3) 对于连续的数值，能用between就不要用in了；再或者使用连接来替换。

### SELECT语句务必指明字段名称

> SELECT*增加很多不必要的消耗（CPU、IO、内存、网络带宽）；增加了使用覆盖索引的可能性；当表结构发生改变时，前断也需要更新。所以要求直接在select后面接上字段名。

### 当只需要一条数据的时候，使用limit 1
> 这是为了使EXPLAIN中type列达到const类型

### 如果排序字段没有用到索引，就尽量少排序

### 如果限制条件中其他字段没有索引，尽量少用or

> or两边的字段中，如果有一个不是索引字段，而其他条件也不是索引字段，会造成该查询不走索引的情况。很多时候使用union all或者是union（必要的时候）的方式来代替“or”会得到更好的效果。

### 尽量用union all代替union

> union和union all的差异主要是前者需要将结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的CPU运算，加大资源消耗及延迟。当然，union all的前提条件是两个结果集没有重复数据。

### 不使用ORDER BY RAND()

> ```order by rand()```随机拿出数据

```sql
select id from 'dynamic' order by rand() limit 1000;
```

上面的SQL语句，可优化为：

```sql
select id from 'dynamic' t1 join(select rand()*(select max(id) from 'dynamic  as nid') t2 on t1.id > t2.nid limit;
```

### 区分in和exists、not in和not exists

```sql
select * from A where id in (select id from B);
```

上面SQL语句相当于

```sql
select * from A where exists (select * from B.id = A.id);
```

> 区分in和exists主要是造成了驱动顺序的改变（这是性能变化的关键），如果是exists，那么以外层表为驱动表，先被访问，如果是IN，那么先执行子查询。所以IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。
关于not in和not exists，推荐使用not exists，不仅仅是效率问题，not in可能存在逻辑问题。如何高效的写出一个替代not exists的SQL语句？
原SQL语句：

```sql
select * from A where a.id not in (select b.id from B);
```

高效的SQL语句：

```sql
select * from A left join B on where A.id = B.id where B.id is null;
```

### 使用合理的分页方式以提高分页的效率

```sql
select * from product limit 866664, 10;
```

> 使用上述SQL语句做分页的时候，可能有人会发现，随着表数据量的增加，直接使用limit分页查询会越来越慢。
优化的方法如下：可以取前一页的最大行数的id，然后根据这个最大的id来限制下一页的起点。比如此列中，上一页最大的id是866612。SQL可以采用如下的写法：

```sql
select * from product where id > 866664 limit 20;
```

### 分段查询

> 在一些用户选择页面中，可能一些用户选择的时间范围过大，造成查询缓慢。主要的原因是扫描行数过多。这个时候可以通过程序，分段进行查询，循环遍历，将结果合并处理进行展示。
如下图这个SQL语句，扫描的行数成百万级以上的时候就可以使用分段查询：

### 避免在where子句中对字段进行null值判断

> 对于null的判断会导致引擎放弃使用索引而进行全表扫描。

### 不建议使用%前缀模糊查询

> 例如LIKE“%name”或者LIKE“%name%”，这种查询会导致索引失效而进行全表扫描。但是可以使用LIKE “name%”。
那如何查询%name%？
如下图所示，虽然给secret字段添加了索引，但在explain结果并没有使用：

> 那么如何解决这个问题呢，答案：使用全文索引。
在我们查询中经常会用到```select id,fnum,fdst from dynamic_201606 where user_name like '%zhangsan%';```这样的语句，普通索引是无法满足查询需求的。庆幸的是在MySQL中，有全文索引来帮助我们。

创建全文索引的SQL语法是：
```sql
alter table dynamic add fulltext index 'id_user_name' ('user_name');
```

使用全文索引的SQL语句是：

```sql
select id,fnum,fdst from dynamic_201606 where match(user_name) against ('zhangsan' in boolean mode);
```

> 注意：在需要创建全文索引之前，请联系DBA确定能否创建。同时需要注意的是查询语句的写法与普通索引的区别。

### 避免在where子句中对字段进行表达式操作
比如：
```sql
select * from A where age*2 = 36;
```

中对字段就行了算术运算，这会造成引擎放弃使用索引，建议改成：

```sql
select * from A where age = 36/2;
```

### 避免隐式类型转换

> where子句中出现column字段的类型和传入的参数类型不一致的时候发生的类型转换，建议先确定where中的参数类型。

### 对于联合索引来说，要遵守最左前缀法则

> 举列来说索引含有字段id、name、school，可以直接用id字段，也可以id、name这样的顺序，但是name;school都无法使用这个索引。所以在创建联合索引的时候一定要注意索引字段顺序，常用的查询字段放在最前面。

### 必要时可以使用force index来强制查询走某个索引

> 有的时候MySQL优化器采取它认为合适的索引来检索SQL语句，但是可能它所采用的索引并不是我们想要的。这时就可以采用forceindex来强制优化器使用我们制定的索引。

### 注意范围查询语句

> 对于联合索引来说，如果存在范围查询，比如between、>、<等条件时，会造成后面的索引字段失效。

### 关于JOIN优化

1. LEFT JOIN 左连接
2. INNER JOIN 内连接
3. RIGHT JOIN 右连接

> LEFT JOIN 左连接，INNER JOIN MySQL会自动找出那个数据少的表作用驱动表，RIGHT JOIN B表为驱动表。
注意：

- MySQL中没有full join，可以用以下方式来解决：
```sql
select * from A left join B on B.name = A.name where B.name is null
union
select * from B;
```

- 尽量使用inner join，避免left join：

> 参与联合查询的表至少为2张表，一般都存在大小之分。如果连接方式是inner join，在没有其他过滤条件的情况下MySQL会自动选择小表作为驱动表，但是left join在驱动表的选择上遵循的是左边驱动右边的原则，即left join左边的表名为驱动表。

### 合理利用索引：

> 被驱动表的索引字段作为on的限制字段。

### 利用小表去驱动大表：


### 巧用STRAIGHT_JOIN：

> inner join是由MySQL选择驱动表，但是有些特殊情况需要选择另个表作为驱动表，比如有group by、order by等「Using filesort」、「Using temporary」时。STRAIGHT_JOIN来强制连接顺序，在STRAIGHT_JOIN左边的表名就是驱动表，右边则是被驱动表。在使用STRAIGHT_JOIN有个前提条件是该查询是内连接，也就是inner join。其他链接不推荐使用STRAIGHT_JOIN，否则可能造成查询结果不准确。

### 比较运算符能用 “=”就不用“<>”

> “=”增加了索引的使用几率。

### 明知只有一条查询结果，那请使用 “LIMIT 1”

> “LIMIT 1”可以避免全表扫描，找到对应结果就不会再继续扫描了。

### 为列选择合适的数据类型

> 能用TINYINT就不用SMALLINT，能用SMALLINT就不用INT，道理你懂的，磁盘和内存消耗越小越好嘛。

### 将大的DELETE，UPDATE or INSERT 查询变成多个小查询

> 能写一个几十行、几百行的SQL语句是不是显得逼格很高？然而，为了达到更好的性能以及更好的数据控制，你可以将他们变成多个小查询。


### 使用UNION ALL 代替 UNION，如果结果集允许重复的话

> 因为 UNION ALL 不去重，效率高于 UNION。

### 为获得相同结果集的多次执行，请保持SQL语句前后一致

> 这样做的目的是为了充分利用查询缓冲。

比如根据地域和产品id查询产品价格，第一次使用了：

```sql
select price from order where id = 3213721 and region = 'beijing';
```

那么第二次同样的查询，请保持以上语句的一致性，比如不要将where语句里面的id和region位置调换顺序。

### 尽量避免使用 “SELECT *”

> 如果不查询表中所有的列，尽量避免使用 SELECT  *，
因为它会进行全表扫描，不能有效利用索引，增大了数据库服务器的负担，以及它与应用程序客户端之间的网络IO开销。

### WHERE 子句里面的列尽量被索引

> 只是“尽量”哦，并不是说所有的列。因地制宜，根据实际情况进行调整，因为有时索引太多也会降低性能。

### JOIN 子句里面的列尽量被索引

> 同样只是“尽量”哦，并不是说所有的列。

### ORDER BY 的列尽量被索引

> ORDER BY的列如果被索引，性能也会更好。

### 使用 LIMIT 实现分页逻辑

> 不仅提高了性能，同时减少了不必要的数据库和应用间的网络传输。

