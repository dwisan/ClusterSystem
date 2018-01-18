```
  เอกสารเรื่อง : การทำ mySQL data Replication รูปแบบ MASTER-MASTER
```
> Requirements
- [x] Static IP address 192.168.15.100
- [x] Static IP address 192.168.15.101
> Install and Configure MySQL on First Master Server
```
apt-get install mysql-server mysql-client
sudo nano /etc/mysql/my.cnf
#
[mysqld_safe]
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
key_buffer_size         = 16M
max_allowed_packet      = 16M
thread_stack            = 192K
thread_cache_size       = 8
myisam-recover-options  = BACKUP
query_cache_limit       = 1M
query_cache_size        = 16M
log_error = /var/log/mysql/error.log

server_id           = 1
bind-address        = 192.168.15.100
log_bin             = /var/log/mysql/mysql-bin.log
log_bin_index       = /var/log/mysql/mysql-bin.log.index
relay_log           = /var/log/mysql/mysql-relay-bin
relay_log_index     = /var/log/mysql/mysql-relay-bin.index
expire_logs_days    = 10
max_binlog_size     = 100M
log_slave_updates   = 1
auto-increment-increment = 2
auto-increment-offset = 1

#

/etc/init.d/mysql restart
mysql_secure_installation

[root@mysqla ~]# mysql -u root -p
mysql> create user 'replication'@'192.168.15.101' identified by 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.15.101';
mysql> FLUSH PRIVILEGES;
mysql> show master status;
```
> Install and Configure MySQL on Second Master Server
```
apt-get install mysql-server mysql-client
nano /etc/mysql/my.cnf

#
[mysqld_safe]
socket          = /var/run/mysqld/mysqld.sock
nice            = 0

[mysqld]
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
key_buffer_size         = 16M
max_allowed_packet      = 16M
thread_stack            = 192K
thread_cache_size       = 8
myisam-recover-options  = BACKUP
query_cache_limit       = 1M
query_cache_size        = 16M
log_error = /var/log/mysql/error.log

server_id           = 2
bind-address        = 192.168.15.101
log_bin             = /var/log/mysql/mysql-bin.log
log_bin_index       = /var/log/mysql/mysql-bin.log.index
relay_log           = /var/log/mysql/mysql-relay-bin
relay_log_index     = /var/log/mysql/mysql-relay-bin.index
expire_logs_days    = 10
max_binlog_size     = 100M
log_slave_updates   = 1
auto-increment-increment = 2
auto-increment-offset = 2
#
/etc/init.d/mysql start
mysql_secure_installation


[root@mysqlb ~]# mysql -u root -p
mysql> create user 'replication'@'192.168.15.100' identified by 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.15.100';
mysql> FLUSH PRIVILEGES;
mysql> show master status;
```
>Configure MySQL Master on Both Server
.
>Configure First Server as Master
```
[root@mysqlb ~]# mysql -u root -p
mysql> SHOW MASTER STATUS;

[root@mysqla ~]# mysql -u root -p
mysql> SLAVE STOP;
mysql> CHANGE MASTER TO master_host='192.168.15.101', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000002', master_log_pos=276; 
mysql> SLAVE START;
```
>Configure Second Server as Master
```
[root@mysqla ~]# mysql -u root -p
mysql> SHOW MASTER STATUS;

[root@mysqlb ~]# mysql -u root -p
mysql> SLAVE STOP;
mysql> CHANGE MASTER TO master_host='192.168.15.100', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000001', master_log_pos=276;
mysql> SLAVE START;
```
