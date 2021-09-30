## 查询数据
- 检索出不同的值
> 使用distinct,```select distinct v-id from Products;```会返回不同的值

- 输出特定的行数
> ```select v-id from Products limit 5 offset 10;```输出结果的第5行起到后面10行数据

- 排序数据
> 使用order by关键字，他应该是select语句中最后一条子句。DESC表示降序，ASC表示升序，默认不写就是升序。

- where子句操作符

操作符| 说明
--|:----
= | 等于
<> | 不等于
!= | 不等于
< | 小于
<= | 小于等于
! | 不小于
> | 大于
>= | 大于等于
!> | 不大于
between | 在指定的两个值之间
is null | 为null的值

- and操作符
> 用来增加多个过滤条件，前后两个条件同时满足

- or操作符
> 用来匹配任一给定的条件

- in操作符
> 用于指定条件范围，范围中每个条件都可以进行匹配

- not操作符
> 否定他后面跟的任何条件

### like操作符

- 百分号（%）通配符
> ```select pro_id from products where name like 'Fish%';```检索名字是Fish开头的行，%表示匹配任意字符出现任意次数。

- 下划线（_）通配符
> 用途和%一样，但是只能匹配单个字符，而不是多个字符。

- 方括号（[ ]）通配符
> 用来指定一个字符集，他必须匹配指定位置的一个字符  ```select * from Customes where name like '[JM]%';```

### 函数

- 文本处理函数UPPER(),可以将文本转换为大写。

### 汇总函数

#### 聚集函数

函数 | 说明
--|:--:|--
AVG()|返回某列的平均值
COUNT()|返回某列的行数
MAX()|返回某列最大值
MIN()|返回某列的最小值
SUM()|返回某列值之和

#### 分组函数

- 创建分组（group by）
> ```selece co_id from products group by co_id;```将数据按照co_id进行分组

- 过滤分组having(having支持所有的select支持操作符)
> where在数据分组前进行过滤，having在数据分组后进行过滤。

#### 分组和排序
> GROUP BY和ORDER BY经常完成相同的工作，但他们非常不同。

ORDER BY | GROUP BY
--|:---
对产生的输出排序 | 对行分组，但输出可能不是分组的顺序
任意列都可以使用 | 只能使用选择列或表达式列，而且必须使用每个选择列表达式
不一定需要 | 如果和聚集函数一起使用，则必须使用

#### select子句顺序
子句 | 说明 | 是否必须使用
--|:---:|---
select | 要返回的列或表达式 | 是 
from | 从中检索数据的表 | 仅在从表选择数据时使用
where | 行级过滤 | 否
group by | 分组说明 | 仅在按组计算聚集时使用
having | 组级过滤 | 否
order by | 输出排序顺序 | 否

### 使用子查询

> 将一个查询结果作为另一个查询语句的条件，子查询总是从内往外处理。

> 只能是单个列，如果是多个列则返回错误。

- 需要使用完全限制名（表名.列名）

### 联结表
> **性能考虑**，联结多张表时，会非常耗费资源，联结的表越多，性能下降约厉害。

- 什么是联结
> 使用特殊的语法，可以联结多个表返回一组输出，联结在运行时关联表中正确的行。

- 创建联结
> ```select ve_name, pro_name, pro_price from vendors, products where vendors.ve_id = products.ve_id;```需要使用完全限制名

- 内联结（inner join）
> ```select ve_name, pro_name, pro_price from vendors inner join products on vendors.ve_id = products.ve_id;```在内联结中，可以用on代替where子句。

### 创建高级联结
1. 可以给表起别名（和给字段起别名语法差不多）
> 通过起不同的别名，多次使用同一张表

2. 可以使用不同类型的联结
> 除了等值联结，还有自联结(self-join)，自然联结(natrual join)和外连接(outer join)。

3. 外联结又分左外联结(left outer join)、右外联结(right outer join)和全外联结(full outer join)

### 组合查询
> 执行多个查询语句，并将结果作为一个结果集返回，这个组合查询也被叫做复核查询。

1. union（加在两条查询语句之间）
> 可以将两条查询语句的结果进行并集运算，得到一个新的结果。（会自动去除重复行）

## 插入数据
1. 插入完整行(需要给每个字段都赋上值)

```mysql
insert into customers values('xxx', 'xxx', 'xx', 'xx', 'xxx');
```

```mysql
insert into customers(cust_id, cust_name, cust_address, cust_city, cust_phone) values('xxx', 'xxx', 'xx', 'xx', 'xxx');
```

2. 插入行的一部分(需要设置好列的默认值)

```mysql
insert into customers(cust_id, cust_name) values('xxx', 'xxx');
```

3. 插入某些查询的结果

```mysql
insert into customers(cust_id, cust_name, cust_address, cust_city, cust_phone) select cust_id, cust_name, cust_address, cust_city, cust_phone from custNew;
```

### 将一个表复制到另一张表中

- select into
```mysql
select * into custCopy from customers;
```
> 创建一张custCopy的新表，并将customers表的整个内容复制过去

- insert select
```mysql
insert into custCopy select * from customers;
```

## 更新数据
1. 更新表中的特定行

```update customers set email = "xxxx@qq.com" where id = "000001";```

2. 更新表中的所有行

```update customers set email = "xxxx@qq.com";```

## 删除数据
1. 删除表中特定行

```delete from customers where id = "00000001";```

2. 删除表中所有行

```delete from customers;```

## 创建和操纵表
> 创建的表名必须不存在否则会报错

1. 创建一张表,not null是值不能为空，不写则默认为null
```mysql
create table products
(
    pro_id     char(10)       not null,
    ven_id     char(10)       not null,
    pro_name   char(254)      not null,
    pro_price  decimal(8,2)   not null,
    pro_desc   varchar(1000)
);
```

2. 可以指定默认值(在后面加上default属性，可以设置默认值)
```mysql
create table order
(
    ord_num    integer        not null,
    quantity    integer        not null     default 1 
);
```

3. 获取系统的日期
```mysql
current_date()
```

4. 更新表
- 将表新增一列
```mysql
alter table order
add ord_state char(20);
```

- 将表删除一列
```mysql
alter table order
drop column ord_state;
```

5. 删除表（删除整个表而不是删除表中的数据）
```mysql
drop table order;
```
