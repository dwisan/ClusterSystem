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
>Configuring the Nodes
```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="this_node_ip"
wsrep_node_name="this_node_name"

```
