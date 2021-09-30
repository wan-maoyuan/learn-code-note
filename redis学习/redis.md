### redis常用数据类型
```
string、 Hash、 List、 Set、 Zset、 Hyper、 loglog
```

## redis的一些其他的功能
1. Redis订阅发布
2. Redis事务--》商品秒杀应用场景
3. Redis数据淘汰策略
4. Redis持久化
5. Redis缓存与数据库同步    使用kafka作为异步队列

### 知识点总结
1. 缓存穿透、缓存雪崩、热点key等处理方案

### Redis高级配置


### Redis的应用场景
1. 限时优惠
2. 网站数据缓存（积分排行版）
3. 手机验证码
4. 限制网站访客访问频率（一分钟最多访问10次）

### string类型
- string是Redis最基本的数据类型，一个key对应一个值
- 是一个二进制安全，不会发生乱码的问题，不涉及编解码。

#### string常用命令
1. 赋值语法  
- 设置给定key的值，如果已经存在，则覆盖旧的值，无视类型
```
set key_name value
```

- 当key不存在时，才可以进行赋值。如果存在，则赋值不成功。
```
//解决分布式锁的方案之一
setnx key value
```

2. 取值语法
- 获取指定key的值，如果key不存在，返回nil，如果只不是字符串类型，返回一个错误。
```
get key_name
```

- 获取指定key的部分值，通过下标的范围。
```
getrange key start end
```

3. 删除语法
- 删除指定的key，如果存在，返回数字类型。
```
del key_name
```

4. 自增/自减
- 将key中存储的数字加上指定的增量值，如果不存在，贤设为0，在执行。
```
incr key_name
incrby key_name 增量值
```

- 将key中存储的数字值减减量值
```
decr key_name
decrby key_name 减量值
```

5. 字符串拼接
- 指定的key追加到末尾，如果不存在，为他赋值。
```
append key_name value
```

### 哈希（hash）数据类型
> 是一个string类型的field和value的映射表，特别适合用于存储对象。可以看成具有key和value的map容器，该类型适合存储值对象信息。

1. 赋值语法
- 为指定的key，设定field/value
```
hset key field value
```

- 同时将多个field/value对设置到哈希表的key中。
```
hmset key field value [field, value]……
```

2. 取值语法
```
hget key field              //获取存储在HASH中的值，根据FIELD得到VALUE
hmget key field[field1]     //获取key所有给定字段的值
hgetall key                 //返回HASH表中所有的字段和值

hkeys key                   //获取所有HASH表中的字段
hlen key                    //获取HASH表中字段的数量 
```

3. 删除语法
```
hdel key field1 [field2]
```

4. 其他语法
- 为HASH表中key中指定字段的整数加上增量
```
hincrby key field increment
```

- 查看HASH表中指定的字段是否存在
```
hexists key field
```

## Redis事务
### 事务命令
1. 取消事务，放弃执行事务内的所有命令
- ```discard```

2. 执行所有事务块内的命令
- ```exec```

3. 标记一个事务块的开始
- ```multi```

4. 取消watch命令对所有key的监视
- ```unwatch```

5. 监视一个或多个key，如果事务执行前这些key被其他命令所改动，事务将被打断。
- ```watch key [key ……]```