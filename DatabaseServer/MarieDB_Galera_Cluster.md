> Infrastructure Setting 
- [x] Ubuntu 18.04 LTS
- [x] MariaDB Galera Cluster {01} : 172.18.111.221 (Galera01)
- [x] MariaDB Galera Cluster {02} : 172.18.111.222 (Galera02)
- [x] MariaDB Galera Cluster {03} : 172.18.111.223 (Galera03)
> Installing MariaDB Database Server On all nodes
```
Galera{01,02,03}# apt update -y
Galera{01,02,03}# apt upgrade -y
Galera{01,02,03}# apt-get install mariadb-server mariadb-client rsync -y
Galera{01,02,03}# systemctl enable mariadb.service
Galera{01,02,03}# mysql_secure_installation

When prompted:

    Enter current password for root (enter for none):  Enter
    Set root password? [Y/n]: Y
    New password: {Enter Password}
    Re-enter new password: {Repeat password}
    Remove anonymous users? [Y/n]: Y
    Disallow root login remotely? [Y/n]: Y
    Remove test database and access to it? [Y/n]: Y
    Reload privilege tables now? [Y/n]: Y

```
>Configuring MariaDB Database Server
- [x] On 172.18.111.221:Galera01 
```
Galera01# nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
#bind-address=127.0.0.1

[galera]

binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="MariaDB_Cluster"
wsrep_cluster_address="gcomm://172.18.111.221,172.18.111.222,172.18.111.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.221"
wsrep_node_name="Galera01"

```
- [x] On 172.18.111.222:Galera02 
```
Galera02# nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
#bind-address=127.0.0.1

[galera]

binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="MariaDB_Cluster"
wsrep_cluster_address="gcomm://172.18.111.221,172.18.111.222,172.18.111.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.222"
wsrep_node_name="Galera02"

```
- [x] On 172.18.111.223:Galera03 
```
Galera03# nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
#bind-address=127.0.0.1

[galera]

binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="MariaDB_Cluster"
wsrep_cluster_address="gcomm://172.18.111.221,172.18.111.222,172.18.111.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.223"
wsrep_node_name="Galera03"

```
> Start and Checking
```
Galera01# systemctl stop mariadb
Galera01# galera_new_cluster
Galera01# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+

Galera02# systemctl stop mariadb
Galera02# systemctl start mariadb
Galera02# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+

Galera03# systemctl stop mariadb
Galera03# systemctl start mariadb
Galera03# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
>Start Order after all down
```
find safe_to_bootstrap =1 in /var/lib/mysql/grastate.dat in all node
and first start that node

# cat /var/lib/mysql/grastate.dat
safe_to_bootstrap =1
# galera_new_cluster

```
