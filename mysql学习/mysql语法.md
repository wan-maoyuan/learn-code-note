## MySQL语法

### 数据库操作
1. 创建数据库

```sql
create database 数据库名;
```

```sql
create database 数据库名 character set 字符集;
```

2. 查看数据库
```sql
show databases;
```

```sql
show create database 数据库名;
```

3. 删除数据库

```sql
drop database 数据库名;
```

4. 其他数据库命令

- 切换数据库： 
```sql
use 数据库名;
```

- 查看正在使用的数据库：
```sql
select database();
```

### 表操作

1. 创建表
```sql
create table 表名(
	字段名 类型（长度）[约束],
	字段名 类型（长度）[约束]
);
```

2. 查看表

- 创建完成之后，我们可以查看数据库的表
```sql
show tables;
```

- 查看表结构
```sql
desc  表名
```

3. 删除表

```sql
drop table 表名
```

4. 修改表
- 添加一列
```sql
alter table 表名 add 字段名 类型（长度）[约束]；
```

- 修改表列的类型长度及约束
```sql
alter table 表名 modify 字段名 类型（长度）[约束]；
```

- 修改表修改列名	
```sql
alter table 表名 change 旧列名 新列名 类型（长度）[约束]；
```

- 修改表删除列
```sql
alter table 表名 drop 列名；
```

- 修改表名
```sql
Rename table 表名 to 新表名
```

- 修改表的字符集
```sql
alter table 表名 character set 字符集；
```

- 插入记录,向列表中插入某些列
```sql
Insert into 表（列名1， 列名2，列名3……）values（值1，值2，值3……）;向表中插入某些列
insert into 表 （值1，值2，值3……）向表中插入所有列
```
