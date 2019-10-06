####  windows下MYSQL服务的安装
##### 1、官网下载MySQL服务安装
##### 2、默认安装完没有配置文件my.ini,需要自己手动创建配置文件
```
mysql安装目录下（与bin目录同级）my.ini
参考配置：
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:\Program Files\MySQL\MySQL Server 5.7
# 设置mysql数据库的数据的存放目录
datadir=C:\Program Files\MySQL\MySQL Server 5.7\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

实例：
[mysql]
default-character-set=utf8
[mysqld]
port=3306
basedir=C:\Program Files (x86)\MySQL\MySQL Server 5.7
datadir=F:\JAVA\mysql\workspace
max_connections=200
character-set-server=utf8
default-storage-engine=INNODB

``` 

##### 3、bin目录下打开cmd，输入命令./mysqld --console，查看缺少mysql.plugin的表
##### 4、手动创建mysql.plugin的表，输入命令./mysqld --initialize --user=mysql --console，此命令会生成一个临时密码
```
A temporary password is generated for root@localhost: ikdfHdqgr8+m

A temporary password is generated for root@localhost: :%LVNlFxS6dF

临时密码是空格之后的部分：
ikdfHdqgr8+m
:%LVNlFxS6dF

```
##### 5、启动mysql服务（net start mysql）
##### 6、进入mysql
```
mysql -u root -p
输入临时密码进入
```
##### 7、修改密码set password = 'password'

(参考地址：https://blog.csdn.net/qq_37449488/article/details/79177153)