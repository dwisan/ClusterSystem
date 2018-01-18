```
เอกสารเรื่อง: การทำ mySQL data Replication รูปแบบ MASTER-MASTER
          (mySQL Data Replication with MASTER-MASTER Mode)
```

> ความต้องการของระบบ (Requirements)
- [x] Server01 : Static IP address 192.168.15.100
- [x] Server02 : Static IP address 192.168.15.101
- [x] OS : Ubuntu 16.04 LTS

> On First Server (Server01)
```
root@Server01:~# apt-get install mysql-server mysql-client

root@Server01:~# sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

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

root@Server01:~# /etc/init.d/mysql restart
```
> On Second Server (Server02)
```
root@Server02:~# apt-get install mysql-server mysql-client
root@Server02:~# nano /etc/mysql/mysql.conf.d/mysqld.cnf

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

root@Server02:~# /etc/init.d/mysql start
```
>Configure Server01 as Master (Replicate data to Server02)
```
root@Server01:~# mysql -u root -p
mysql> create user 'replication'@'192.168.15.101' identified by 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.15.101';
mysql> FLUSH PRIVILEGES;
```
>Configure Server02 as Master (Replicate data to Server01)
```
root@Server02:~# mysql -u root -p
mysql> create user 'replication'@'192.168.15.100' identified by 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.15.100';
mysql> FLUSH PRIVILEGES;
```
>Configure Server01 as Slave
```
root@Server02:~# mysql -u root -p
mysql> SHOW MASTER STATUS;

root@Server01:~# mysql -u root -p
mysql> STOP SLAVE;
mysql> CHANGE MASTER TO master_host='192.168.15.101', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000002', master_log_pos=276; 
mysql> START SLAVE;
```
>Configure Server02 as Slave
```
root@Server01:~# mysql -u root -p
mysql> SHOW MASTER STATUS;

root@Server02:~# mysql -u root -p
mysql> STOP SLAVE;
mysql> CHANGE MASTER TO master_host='192.168.15.100', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000001', master_log_pos=276;
mysql> START SLAVE;
```
