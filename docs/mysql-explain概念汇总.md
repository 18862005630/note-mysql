# mysql-explain概念汇总

## explain基本概念
### 1、explain定义
```
mysql中的查询优化器Query Optimizer作用是找到最小代价的正确执行方案。
Explain是模拟mysql优化器执行sql查询语句，分析查询语句或表结构的性能瓶颈。
```

### 2、explain作用
在sql语句前加上explain，可以看出：
```
（1）表的读取顺序id
（2）数据读取操作的操作类型（select_type）
（3）哪些索引可以使用（possible_keys）
（4）哪些索引被实际使用（key）
（5）表之间的引用（ref）
（6）每张表有多少行被优化器查询（rows）
```

### 3、explain执行计划包含的信息
```
id:表读取顺序
select_type:数据读取的操作类型
table:哪张表
partitions:分区表中的命中情况，无分区表则为null
type:查询时使用了哪种查询类型，一般常见7种，由好到坏（system > const > eq_ref > ref > range > index > All）,一般保证查询达到range，最好到ref，避免是all
possible_keys:所查询的字段如果存在索引，全部列出，未必实际使用
key:查询中实际使用的索引
key_len:使用到的索引长度，仅会计算where条件中应用到的索引长度，order_by和group_by中的不会计算
ref:条件字段类型，常量显示const，关联查询显示关联字段，若应用表达式或函数处理则显示func
rows:索引查询时大致估算要查询的行数，rows越小越好
extra:额外信息，包含三种（1、using filesort 2、using temporary 3、using index）
1、using filesort
建立的索引未被使用，执行了文件排序，可能sql语句有问题或建立的索引有冲突

2、using temporary
使用了临时表来保存中间结果，说明建立的索引没有使用完全，常见于剖排序order by和分组查询group by

3、using index
查询操作中使用了覆盖索引(根据索引字段查询索引字段，即where子句条件是索引字段，select查询的也是索引字段)，表示sql执行效率不错
若此时同时出现using where，表明索引被用来执行索引键值的查找了
若此时没有出现using where，表明索引只是被用来读取数据，并没有用来执行索引键值的查找
```

#### sql语句问题导致索引失效的情况：
```
1、索引字段若进行了表达式或函数处理，索引是不会被应用的
2、where子句中有null判断，引擎会放弃使用索引而是用全表查询（表字段尽量给默认值，避免留null）
3、where子句中出现！=或<>判断也会导致引擎放弃使用索引查询
4、or查询也会导致引擎放弃使用索引查询
5、in或not in也会导致引擎放弃使用索引查询，范围查询尽量用between，某些情况用exists代替in
6、模糊查询（只有单个通配符%在字符右侧时可使用索引，字段为字符型，日期类型使用like不影响使用索引），即使用like会导致引擎放弃使用索引查询而进行全表查询
7、组合索引时，where子句顺序尽量按索引字段顺序(顺序倒了不会导致不使用索引，只是执行中比顺序执行耗些性能)，且组合索引的第一个字段必须出现在where子句中（会变成全表查询all），否则不会使用索引(遵循左前缀保留原则)
```

