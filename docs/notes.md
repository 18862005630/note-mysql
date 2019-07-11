# MYSQL NOTE
1、!=是比较定值，在条件筛选的时候先排除null的情况再筛选，所以null的数据不会被筛选出来
```mysql [{'name':'zyx','score':NULL},{'name':'ch','score':90},{'name':'sj','score':100}]
	select name from user where score !=100;
```
此查询结果只有ch，zyx不会被筛选出来。

2、distinct去重是针对查询所有字段，只有当查询的所有字段同时相同时才去重
``` 
select distinct name,score from user;
```
只有当name和score都相同时才去重，若只是name相同或者score相同是不去重的。

3、limit限制返回结果

``` 
	select * from user limit 5;
	select * from user limit 4,5;//从第四条开始往后筛选5条
	select * from user limit 4 offset 5;
``` 

4、order by按字段排序，默认升序asc，倒序为desc
```
	select score,name from user order by score,name;//可多字段排序，优先按score升序，若score相同，则按照name升序A-Z
```

5、and和or的使用，最好用括号（）明确条件筛选顺序，而不是依赖于操作符的优先级
```
select * from user where score=100 or score=90 and sex=male;
//此语句若没有用（）明确处理顺序，那么会按照and和or的优先级进行处理，and优先级最高，所以先处理and两侧的。等价于：
select * from user where score=100 or (score=90 and sex=male);
//这就与我们实际想筛选的不一致了，我们实际想筛选男生里面，分数等于100或等于90的学生，可sql语句变成了筛选分数等于100的学生或男生里面分数等于90的学生
```

6、in和or语义相同，但推荐优先使用in，除了in语句更加简洁直观，执行效率更快外，in还能包含其他select子句

7、其他DBMS允许not对各种条件取反，而mysql中not只对in，between，exists有效

8、聚合函数里可以配合distinct使用，如avg（distinct score），去重后取平均值

9、过滤分组，可以使用where+group by也可以使用group by+having，优先推荐where子句方式，效率高
```
select id,name from user where id! = 1 group by id;
等价于
select id,name from user group by id having id != 1;
```
但是having比where更容易解决复杂问题：
```
select id,name,sum(score) from user group by id having sum(score)>100;
```

10、select 子句书写顺序：
select 
from 
where 
group by 
having
order by
limit 

11、子查询
- 子查询的数目虽然没有限制，但实际受性能影响，不能嵌套太多子查询
- 保证where子句中的列与子查询select中的列相同
- 子查询一般与in操作符结合使用

12、全文本搜索fulltext，字段类型
```
select notes from article where Match(notes) Against('abc');
文章中查询带abc字符串的文本行，不区分大小写

select notes,Match(notes) Against('abc') as rank from article;
查询文章每行中带有字符串abc的等级rank，无为0，有如果靠行前值就越大
```

