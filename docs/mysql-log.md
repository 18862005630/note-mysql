#数据库中日志监听步骤
1、进入Mysql配置文件 /etc/mysql/mysql.conf.d/mysqld.cnf
2、将genral_log_file和general_log两行解除注解#
3、重启服务
service mysql restart
4、实时监听日志
tail -f /var/log/mysql/mysql.log

