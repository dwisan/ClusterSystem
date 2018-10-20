> MariaDB Galera Cluster - Known Limitations
```
- [x] Storage Engine : innoDB Only to Replicated

ref.
https://mariadb.com/kb/en/library/mariadb-galera-cluster-known-limitations/

```
> Infrastructure Setting 
- [x] MariaDB Galera Cluster {01} : 172.18.111.61 (mrdb-cls01)
- [x] MariaDB Galera Cluster {02} : 172.18.111.62 (mrdb-cls02)
- [x] MariaDB Galera Cluster {03} : 172.18.111.63 (mrdb-cls03)

> Node Specifications
- [x] Motherboard : P5L-MX/IPAT
- [x] CPU : Intel(R) Core(TM)2 Duo CPU     E4500  @ 2.20GHz
- [x] RAM : 2 x 1G DIMM DDR2 Synchronous 667 MHz
- [x] Hdd : ATA Disk 80GB 5400rpm 
- [x] OS  : Ubuntu 18.04 LTS 64bit
- [x] FS  : Brtfs
- [x] Performance Testing:
```
  CPU Benchmark
  # sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
  # sysbench --num-threads=2 --test=cpu --cpu-max-prime=200000 run
  result: 
      mrdb-cls01 : total time 10.0224s
      mrdb-cls02 : total time 10.0139s
      mrdb-cls03 : total time 10.0138s
  
  Memory Benchmark
  - [x] Write Testing
  # sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
  # sysbench --test=memory --memory-block-size=1k --memory-oper=write --threads=2 run
  result:
      mrdb-cls01 : 3480.97 MiB/sec
      mrdb-cls02 : 3471.33 MiB/sec
      mrdb-cls03 : 3485.08 MiB/sec
  
  - [x] Read Testing    
  # sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
  # sysbench --test=memory --memory-block-size=1k --memory-oper=read --threads=2 run
  result:
      mrdb-cls01 : 4635.13 MiB/sec
      mrdb-cls02 : 4687.52 MiB/sec
      mrdb-cls03 : 4469.18 MiB/sec
      
  File IO Benchmark
  # sysbench --test=fileio --file-total-size=20G --file-num=10 --threads=2 prepare
  # sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
  # sysbench --test=fileio --file-total-size=20G --file-num=10 --file-test-mode=rndrw --time=300 --max-requests=0 --threads=2 run
  result: random r/w test
      mrdb-cls01 : Write: 0.56 MiB/sec  Read: 0.85 MiB/sec
      mrdb-cls02 : Write: 0.56 MiB/sec  Read: 0.84 MiB/sec
      mrdb-cls03 : Write: 0.56 MiB/sec  Read: 0.84 MiB/sec
  
  # sysbench --test=fileio --file-total-size=20G cleanup
```
> Installing MariaDB Database Server On all nodes
```
mrdb-cls{01,02,03}# apt update -y
mrdb-cls{01,02,03}# apt upgrade -y
mrdb-cls{01,02,03}# apt-get install mariadb-server mariadb-client rsync -y
mrdb-cls{01,02,03}# systemctl enable mariadb.service
mrdb-cls{01,02,03}# mysql_secure_installation

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
- [x] On 172.18.111.61 (mrdb-cls01)
```
mrdb-cls01# nano /etc/mysql/mariadb.conf.d/50-server.cnf

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
wsrep_cluster_address="gcomm://172.18.111.61,172.18.111.62,172.18.111.63"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.61"
wsrep_node_name="mrdb-cls01"

```
- [x] On 172.18.111.62 (mrdb-cls02)
```
mrdb-cls02# nano /etc/mysql/mariadb.conf.d/50-server.cnf

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
wsrep_cluster_address="gcomm://172.18.111.61,172.18.111.62,172.18.111.63"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.62"
wsrep_node_name="mrdb-cls02"

```
- [x] On 172.18.111.63 (mrdb-cls03)
```
mrdb-cls03# nano /etc/mysql/mariadb.conf.d/50-server.cnf

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
wsrep_cluster_address="gcomm://172.18.111.61,172.18.111.62,172.18.111.63"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="172.18.111.63"
wsrep_node_name="mrdb-cls03"

```
> Start and Checking
```
- [x] Starting on mrdb-cls01
mrdb-cls01# systemctl stop mariadb
mrdb-cls01# galera_new_cluster
mrdb-cls01# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+

- [x] Starting on mrdb-cls02
mrdb-cls02# systemctl stop mariadb
mrdb-cls02# systemctl start mariadb
mrdb-cls02# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+

- [x] Starting on mrdb-cls03
mrdb-cls03# systemctl stop mariadb
mrdb-cls03# systemctl start mariadb
mrdb-cls03# mysql -uroot -p -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size'"

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
>Starting Order after all nodes down
```
-[Method-01] 

find "safe_to_bootstrap =1" in /var/lib/mysql/grastate.dat on all node
and starting first that node

# cat /var/lib/mysql/grastate.dat
safe_to_bootstrap =1
# galera_new_cluster

-[Method-02] starting with lastest shutdown node 
```
> Performance Testing

```
# mysql -uroot -p
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password';
MariaDB [(none)]> create database sbtest;

# apt install sysbench
>> test io write and checking data synchornized between all nodes 
# sysbench /usr/share/sysbench/oltp_read_only.lua --db-driver=mysql --threads=1 --mysql-host=172.18.111.221 --mysql-user=root --mysql-password=password --mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=1000000 prepare
# sysbench /usr/share/sysbench/oltp_read_only.lua --db-driver=mysql --threads=1 --events=0 --time=300 --mysql-host=172.18.111.221 --mysql-user=root --mysql-password=pass --mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=10000000 --range_selects=off --db-ps-mode=disable --report-interval=1 run
```

