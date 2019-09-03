#### mysql-创建及授权

#### 创建数据库
```
create databases zyx default character set utf8 collate utf8_general_ci;
```

#### 创建用户
```
create user 'ch'@'localhost' identified by '123456';
```

#### 给用户授权
```
grant all privileges on zyx.* to 'ch'@'localhost' identified by '123456';

grant all on zyx.* to ch@'localhost';
```

#### 查询某用户的权限
```
show grants for ch@'localhost';
```

#### 查询mysql中所有用户信息
```
select user,host from mysql.user;
```