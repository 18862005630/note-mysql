# mysql-explain
sql命令前面加上explain会输出以下几种属性特征：

### 1、id
sql语句的执行顺序

### 2、select_type
sql语句的类型

2、1：simple
表示简单的select或者多表查询，没有union和子查询
如：
```
select * from users;
select * from users as a,user_info as b where a.id=b.user_id and a.age>18;
select * from users as a left join user_info as b on a.id=b.user_id where a.age>18;
```

2、2：primary
有子查询的语句中最外层的select查询就是primary
如：
```
explain select * from (select * from users limit 10) as u;
```

2、3：union
union语句中的后一条语句的类型就是union
如：(union后的语句select * from users limit 10,10)
```
explain select * from users limit 10 union select * from users limit 10,10;
```

2、4：dependent union
子查询中有union语句并且与外层查询有依赖关系，则子查询中union后的语句类型属于dependent union
如：(union后的语句select id from users where name='zyx')
```
explain select * from users where id in (select id from user where name='ch' union select id from users where name='zyx');
```

2、5：union result
union操作的结果，通常id为null

2、6：subquery
子查询中首个select
如：(select id from users where age=18)
```
select * from user_info where user_id = (select id from users where age=18);
```

注意：若=改为in则是simple类型
如:
```
select * from user_info where user_id in (select id from users where age=18);
```

2、7：dependent subquery(此类型会严重消耗性能)
子查询中首个select且与外层查询有依赖关系
如：(select id from user where name='ch')
```
explain select * from users where id in (select id from user where name='ch' union select id from users where name='zyx');
```

2、8：DERIVED（mysql5.7开始做了优化，子查询与父查询合并进行直接查询，类型为simple）
被驱动的子查询，子查询在from之后where之前，外层查询从临时表中获取数据
如：(select * from users,结果集形成临时表a)
```
select name from (select * from users) a where a.age=18;
```

### 3、table
输出行所用的表

### 4、partitions
partitions代表分区表中的命中情况，非分区表，该项为null

### 5、type（表示查询方式的好坏）
##### 以下从最佳类型到最差类型排序
5、1：system
表仅有一行，这是const类型的特例，平时不会出现，可以忽略不计

5、2：const(直接使用primary key 或unique查询)
表最多有一个匹配行，一定是用到primary key或unique才会出现const
如：(id为主键，所以type为const)
```
select * from users where id = 1;
```

错误例子：(虽然也只有一个匹配行，但没有用到primary key 或unique)
```
select * from users limit 1;
```

5、3：eq_ref（联表查询使用primary key 或unique查询）
mysql5.7
如：
a表主键id，唯一且不为空索引uid
b表主键id，唯一且不为空索引uid
```
以下可出现eq_ref类型
explain select a.id,b.id from a,b where a.uid = b.uid;
explain select a.uid,b.uid from a,b where a.uid = b.uid;

explain select * from user_info where user_id in(select id from users);
```

```
以下不行，虽然网上资料显示可以
explain select * from a,b where a.uid = b.uid;
此会变成all类型
```

5、4：ref（使用普通索引查询）
如：users表中email做普通索引
```
explain select * from users where email = '1232@qq.com';
```

5、5：fulltext
全文索引检查，此优先级很高，同时存在全文索引和普通索引的时候，mysql会优先使用全文索引

5、6：ref_or_null
与ref类似，增加了对null值的搜索

5、7：index_merge
表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所有索引，性能可能大部分时间都不如range

5、8：unique_subquery
用于where中的in形式子查询，子查询返回不重复值唯一值。
如：value IN (SELECT primary_key FROM single_table WHERE some_expr)

5、9：index_subquery
用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。
如：value IN (SELECT key_column FROM single_table WHERE some_expr)

5、10：range（索引范围扫描）
常见于使用 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN()或者like等运算符的查询中

5、11：index（索引全表扫描）
把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询

5、12：all（全表扫描数据文件）

### 6、possible_keys
查询中可能使用到的索引

### 7、key
查询中真正使用到的索引(当select_type为index_merge时，这里可能出现两个以上的索引，其他只会出现一个索引)

### 8、key_len
MySQL会用到的索引长度，单列索引用到整个索引的长度，index_merge中算出具体用到的索引长度，该字段只会算where条件用用到索引，order by和group by不在统计范围内

### 9、ref
如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

### 10、rows
执行计划中估算的扫描行数，非精确值，值越大越不好

### 11、filtered
5.7之后的版本默认就有这个字段，不需要使用explain extended了。这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数

### 12、extra
查询计划器给出的额外信息说明
若extra中出现using filesort和using temporary就说明语句需要优化了，此两项非常消耗性能



