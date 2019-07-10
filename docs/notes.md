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

