# mysql-explain
sql命令前面加上explain会输出以下几种属性特征：

###### 1、id
sql语句的执行顺序

###### 2、select_type
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

###### 3、table
输出行所用的表

###### 4、type（表示查询方式的好坏）
连接类型，以下从最佳类型到最差类型排序
4、1：system
表仅有一行，这是const类型的特例，平时不会出现，可以忽略不计

4、2：const(直接使用primary key 或unique查询)
表最多有一个匹配行，一定是用到primary key或unique才会出现const
如：(id为主键，所以type为const)
```
select * from users where id = 1;
```

错误例子：(虽然也只有一个匹配行，但没有用到primary key 或unique)
```
select * from users limit 1;
```

4、3：eq_ref（联表查询使用primary key 或unique查询）
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

4、4：ref（使用普通索引查询）
如：users表中email做普通索引
```
explain select * from users where email = '1232@qq.com';
```

4、5：fulltext
全文索引检查，此优先级很高，同时存在全文索引和普通索引的时候，mysql会优先使用全文索引

4、6：ref_or_null
与ref类似，增加了对null值的搜索

4、7：index_merge
表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取所有索引，性能可能大部分时间都不如range

4、8：unique_subquery
用于where中的in形式子查询，子查询返回不重复值唯一值。
如：value IN (SELECT primary_key FROM single_table WHERE some_expr)




