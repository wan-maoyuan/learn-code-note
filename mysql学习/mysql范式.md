## 数据库的三大设计范式
1. 数据库中的所有字段都是不可分割的原子值。
> 可以拆分字段

2. 满足第一范式的前提下，除了主键外每一列都必须完全依赖主键。（联合主键除外）。
> 可以进行拆表

3. 必须先满足第二范式。除了主键列，其他列之间不能有传递依赖关系。

### 主键约束
> 它能够唯一确定一张表中的一条记录，也就是我们通过给某个字段添加约束，就可以使得该字段不重复也不为空。

```sql
create table user(
    id  int primary key,
    name varchar(20)
);
```
### 联合主键
> 通过给多个字段添加约束，只要这些字段中的任何一个不重复且不为空即可。

```sql
create table user(
    id  int,
    name varchar(20),
    primary key(id, name)
);
```

### 自增约束
> 通过给字段自增约束，保证数值变化不重复。

```sql
create table user(
    id  int primary key auto_increment,
    name varchar(20)
);
```

### 非空约束
> 约束修饰的字段的值不可以为空

```sql
create table user(
    id  int,
    name varchar(20) not null
);
```

### 默认约束
> 当插入数据时，该字段没有传值，就会使用设置好的默认值

```sql
create table user(
    id  int,
    name varchar(20),
    age int default 10
);
```

### 外键约束
> 涉及到两张表，父表和子表。

> 主表中没有的数据值，在子表中是不可以使用的。

> 当主表中的记录被子表引用，是不可以删除的。

```sql
create table classes(
    id  int primary key,
    name varchar(20)
);

create table students(
    id int primary key,
    name varchar(20),
    class_id int,
    foreign key(class_id) references classes(id)
);
```

### explain
- type
> 性能排序：system > const > eq_ref > ref > range > index > ALL