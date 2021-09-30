# MySQL数据库高级(centos环境下)

## 一、前期准备工作

1. 检查系统中是否已经安装过MySQL

   ```shell
   # 使用rpm检查是否安装过MySQL
   rpm -qa | grep mysql
   ```

2. 安装MySQL-server和MySQL-client

   > 安装包需要放在```/opt```目录下

   ```shell
   rpm -ivh mysql-server*.rpm
   rpm -ivh mysql-client*.rpm
   # 安装完成之后，检查是否安装成功
   mysql --version
   ```

3. 启动MySQL

   ```shell
   # 启动MySQL
   service start mysql
   # 关闭MySQL
   service stop mysql
   # 查看MySQL是否启动成功
   ps -ef | grep mysql
   ```

4. 修改MySQL密码

   > mysql 刚装上是默认没有密码

   ```shell
   /usr/bin/mysqladmin -u root password "your password"
   ```

5. 设置MySQL开机自启

   ```shell
   chkconfig mysql on
   ```

6. MySQL的安装位置

   > 在```/var/lib/mysql``` 文件夹中

   | 路径                    | 解释                      | 备注                                    |
   | ----------------------- | ------------------------- | --------------------------------------- |
   | ```/var/lib/mysql/```   | MySQL数据库文件的存放路径 | ```/var/lib/mysql/atguigu.cloud.pid```  |
   | ```/usr/share/mysql/``` | 配置文件目录              | ```mysql.server```命令及配置文件        |
   | ```/usr/bin```          | 相关命令目录              | ```mysqladmin```、```mysqldump```等命令 |
   | ```/etc/init.d/mysql``` | 启停相关脚本              |                                         |

## 二、mysql 配置文件

1. 二进制日志文件```log-bin```

   > 主要用来主从复制

2. 错误日志```log-error```

   > 默认是关闭的，记录严重警告和错误信息，每次启动和关闭信息

3. 查询日志```log```

   > 默认关闭，记录查询的sql语句，开启会降低MySQL整体的性能

4. 数据文件

   > 默认路径```/var/lib/mysql/```

   > ```***.frm```文件存放表结构
   >
   > ```***.myD```文件存放数据
   >
   > ```***.myI```文件存放索引

## 三、MySQL存储引擎

| 对比项   | MyISAM                                             | InnoDB                                     |
| -------- | -------------------------------------------------- | ------------------------------------------ |
| 主外键   | 不支持                                             | 支持                                       |
| 事务     | 不支持                                             | 支持                                       |
| 行表锁   | 表锁，即使操作一条记录也会锁住整个表，不适合高并发 | 行锁，操作时只锁定一行，适合高并发         |
| 缓存     | 只缓存索引，不缓存数据                             | 不仅缓存索引，还缓存数据，对内存要求比较高 |
| 表空间   | 小                                                 | 大                                         |
| 关注点   | 性能                                               | 事务                                       |
| 默认安装 | Y                                                  | Y                                          |

## 四、性能下降SQL变慢

1. 查询语句写的太烂
2. 索引失效
3. 关联查询太多join
4. 服务器调优及各个参数设置

## 五、七种SQL join理论

![sql-join](..\imgs\sql-join.png)

**1. LEFT  JOIN 左连接**

> 结果为左表的全部内容

**2. RIGHT  JOIN右连接**

> 结果为右表的全部内容

**3. 左表独占**

> 左表去除右表中有的数据

**4. INNER  JOIN内连接**

> 筛选出左右表共有的数据

**5. 右表独占**

> 右表去除左表中有的数据

**6. FULL  OUTER  JOIN全连接**

> 结果为左右表所有的数据

**7. 左右表各自独有**

> 左右表去除共有的数据，结果为各自所独有的数据

## 六、索引

**1. 什么是索引**

> - 索引是帮助MySQL高效获取数据的数据结构。排好序的快速查找数据结构。
>
> - 索引的目的在于提高效率，可以类比字典
>
> - 一般来说索引很大，不可能全部存储在内存中，往往以文件的形式存储在硬盘上

**2. 优势**

> - 提高数据检索的效率，降低数据库的IO成本
>
> - 降低数据排序的成本，降低了CPU的消耗

**3. 劣势**

> - 索引也是一张表，保存了主键和索引字段，也要占用磁盘空间
> - 索引虽然提高了查询速度，但是会降低表的更新速度

**4.索引分类**

- 单值索引

  > 一个索引只包含了单个列，一张表可以有多个单列索引

- 唯一索引

  > 索引列的值必须唯一，但允许有空值

- 复合索引

  > 一个索引包含了多个列

- 基本语法

  ```mysql
  # 创建 UNIQUE:表示唯一索引
  CREATE [UNIQUE] INDEX indexName ON mytable(columnname(length));
  ALTER mytable ADD [UNIQUE] INDEX [indexName] ON (columnname(length));
  # 删除
  DROP INDEX [indexName] ON mytable;
  # 查看
  SHOW INDEX FROM table_name; 
  
  # 1.该语句添加一个主键，这意味着索引值必须唯一，且不能为NULL
  ALTER TABLE tableName ADD PRIMARY KEY (column_list);
  # 2.这条语句创建索引的值必须是惟一的（除了NULL外，NULL可能会出现多次）
  ALTER TABLE tableName ADD UNIQUE index_name(column_list);
  # 3.添加普通索引，索引值可能出现多次
  ALTER TABLE tableName ADD INDEX index_name(column_list);
  # 4.该语句指定了索引为FULLTEXT，用于全文索引。
  ALTER TABLE tableName ADD FULLTEXT index_name(column_list);
  ```

**5. 索引结构**

- BTree索引
- Hash索引
- full-text全文索引
- R-Tree索引

**6. 哪些情况下需要建立索引**

- 主键自动创建唯一索引
- 频繁作为查询条件的字段应该创建索引
- 查询中与其他表关联的字段，外键关系建立索引
- 频繁更新的字段不适合创建索引
- where条件里用不到的字段不要创建索引
- 在高并发下建议创建组合索引
- 查询中统计或者分组字段

**7. 哪些情况不要建立索引**

- 表记录太少
- 经常增删改的表
- 数据大量重复且分布均匀的表字段

## 性能分析

**1. MySQL常见性能瓶颈**

- CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据的时候
- IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
- 服务器硬件的性能瓶颈：top,free,iostat和vmstat来查看系统的性能状态

**2. explain执行计划**

- 是什么？

  > EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL如何处理你的SQL语句。

- 能干嘛？

  > 1. 表的读取顺序
  > 2. 数据读取操作的操作类型
  > 3. 哪些索引可以使用
  > 4. 哪些索引被实际使用
  > 5. 表之间的引用
  > 6. 每张表有多少行被优化器查询

- 怎么用？

  > expain + SQL语句
  >
  > 执行计划包含的信息

  | id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
  | ---- | ----------- | ----- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |

  
