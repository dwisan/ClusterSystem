> Requirements
- [x] Static IP address 192.168.15.100
- [x] Static IP address 192.168.15.101
> Install and Configure MySQL on First Master Server
```
apt-get install mysql-server mysql-client
sudo nano /etc/mysql/my.cnf
#
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

mysql -u root -p
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
mysql -u root -p

mysql -u root -p
mysql> create user 'replication'@'192.168.15.100' identified by 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.15.100';
mysql> FLUSH PRIVILEGES;
mysql> show master status;
```
>Configure MySQL Master on Both Server
.
>Configure First Server as Master
```
mysql -u root -p
mysql> SHOW MASTER STATUS;
mysql> SLAVE STOP;
mysql> CHANGE MASTER TO master_host='192.168.15.101', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000002', master_log_pos=276; 
mysql> SLAVE START;
```
>Configure Second Server as Master
```
mysql -u root -p 
mysql> SHOW MASTER STATUS;
mysql> SLAVE STOP;
mysql> CHANGE MASTER TO master_host='192.168.15.100', master_port=3306, master_user='replication', master_password='password', master_log_file='mysql-bin.000001', master_log_pos=276;
mysql> SLAVE START;
```
