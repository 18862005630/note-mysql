# MYSQL NOTE
1、!=是比较定值，在条件筛选的时候先排除null的情况再筛选，所以null的数据不会被筛选出来
```mysql [{'name':'zyx','score':NULL},{'name':'ch','score':90},{'name':'sj','score':100}]
	select name from user where score !=100;
```
此查询结果只有ch，zyx不会被筛选出来。
