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
> Preparing the Server
```
[x] Disabling SELinux for mysqld
    # semanage permissive -a mysqld_t
[x] Firewall Configuration
    # iptables --append INPUT --protocol tcp \
      --source xxx.xxx.xxx.111 --jump ACCEPT
[x] Disabling AppArmor
    # ln -s /etc/apparmor.d/usr /etc/apparmor.d/disable/.sbin.mysqld
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
# -- Ensure that mysqld is not bound to 127.0.0.1
#bind-address=127.0.0.1
bind-address=0.0.0.0

[galera]

# --- Ensure that the binary log format is set to use row-level replication, as opposed to statement-level replication.
# --- Do not change this value, as it affects performance and consistency. The binary log can only use row-level replication
binlog_format=row

# --- Ensure that the default storage engine is InnoDB
# --- Galera Cluster will not work with MyISAM or similar nontransactional storage engines
default_storage_engine=InnoDB

# --- Ensure that the InnoDB locking mode for generating auto-increment values is set to interleaved lock mode.
# --- Do not change this value. Other modes may cause INSERT statements on tables with AUTO_INCREMENT columns to fail.
innodb_autoinc_lock_mode=2

# --- Ensure that the InnoDB log buffer is written to file once per second, rather than on each commit, to improve performance.
innodb_flush_log_at_trx_commit=0

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
bind-address=0.0.0.0

[galera]

binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_flush_log_at_trx_commit=0

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
bind-address=0.0.0.0

[galera]

binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_flush_log_at_trx_commit=0

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
- [x] Preparing 
# apt install sysbench
# mysql -uroot -p
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password';
MariaDB [(none)]> create database sbtest;

# sysbench /usr/share/sysbench/oltp_read_only.lua --db-driver=mysql --threads=2 --mysql-host={host} --mysql-user={user} --mysql-password={password}--mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=1000000 prepare

- [x] Testing Read/Write

Read Only:
# sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
# sysbench /usr/share/sysbench/oltp_read_only.lua --db-driver=mysql --threads=2 --events=0 --time=300 --mysql-host={host} --mysql-user={user} --mysql-password={password} --mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=10000000 --report-interval=1 run

  result:
      mrdb-cls01 : transactions: 524.94 per sec. 
                   queries: 8399.04 per sec.
      mrdb-cls02 : transactions: 527.09 per sec.
                   queries: 8433.47 per sec.
      mrdb-cls03 : transactions: 539.30 per sec. 
                   queries: 8628.73 per sec.

write Only:
# sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
# sysbench /usr/share/sysbench/oltp_write_only.lua --db-driver=mysql --threads=2 --events=0 --time=300 --mysql-host={host} --mysql-user={user} --mysql-password={password} --mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=10000000 --report-interval=1 run

  result:
      mrdb-cls01 : transactions: 7.60 per sec.
                   queries: 45.61 per sec.
      mrdb-cls02 : transactions: 8.07 per sec.
                   queries: 48.43 per sec.
      mrdb-cls03 : transactions: 7.41 per sec.
                   queries: 44.44 per sec.

Read-Write:
# sync; echo 3 > /proc/sys/vm/drop_caches && swapoff -a && swapon -a
# sysbench /usr/share/sysbench/oltp_read_write.lua --db-driver=mysql --threads=2 --events=0 --time=300 --mysql-host={host} --mysql-user={user} --mysql-password={password} --mysql-port=3306 --mysql_storage_engine=innodb --tables=1 --table-size=10000000 --report-interval=1 run

  result:
      mrdb-cls01 : transactions: 7.55 per sec.
                   queries: 150.95 per sec.
      mrdb-cls02 : transactions: 7.79 per sec.
                   queries: 155.82 per sec.
      mrdb-cls03 : transactions: 7.84 per sec.
                   queries: 156.87 per sec.
```
> Tunning and improvement
```
max_connections = 1000
skip-name-resolve=1
query_cache_size = 0

# files
innodb_log_file_size=1024M

# Monitoring
performance_schema=0

# Buffer
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=2
innodb_log_buffer_size=128M
innodb_change_buffering=all
innodb_change_buffer_max_size=25

innodb_flush_log_at_trx_commit=1
skip-innodb_doublewrite

```
> MySQL Options for OLTP RO and Point SELECT tests:
```
# general
table_open_cache = 8000
table_open_cache_instances=16
back_log=1500
query_cache_type=0
max_connections=4000
 
# files
innodb_file_per_table
innodb_log_file_size=1024M
innodb_log_files_in_group=3
innodb_open_files=4000
 
# Monitoring
innodb_monitor_enable = '%'
performance_schema=OFF #cpu-bound, matters for performance
 
#Percona Server specific
userstat=0
thread-statistics=0
 
# buffers
innodb_buffer_pool_size=128000M
innodb_buffer_pool_instances=128 #to avoid wait on InnoDB Buffer Pool mutex
innodb_log_buffer_size=64M
 
# InnoDB-specific
innodb_checksums=1 #Default is CRC32 in 5.7, very fast
innodb_use_native_aio=1
innodb_doublewrite= 1 #https://www.percona.com/blog/2016/05/09/percona-server-5-7-parallel-doublewrite/
innodb_stats_persistent = 1
innodb_support_xa=0 #(We are read-only, but this option is deprecated)
innodb_spin_wait_delay=6 #(Processor and OS-dependent)
innodb_thread_concurrency=0
join_buffer_size=32K
innodb_flush_log_at_trx_commit=2
sort_buffer_size=32K
innodb_flush_method=O_DIRECT_NO_FSYNC
innodb_max_dirty_pages_pct=90
innodb_max_dirty_pages_pct_lwm=10
innodb_lru_scan_depth=4000
innodb_page_cleaners=4
 
# perf special
innodb_adaptive_flushing = 1
innodb_flush_neighbors = 0
innodb_read_io_threads = 4
innodb_write_io_threads = 4
innodb_io_capacity=2000
innodb_io_capacity_max=4000
innodb_purge_threads=4
innodb_max_purge_lag_delay=30000000
innodb_max_purge_lag=0
innodb_adaptive_hash_index=0 (depends on workload, always check)
```
> MySQL Options for OLTP RW:
```
#Open files
table_open_cache = 8000
table_open_cache_instances = 16
query_cache_type = 0
join_buffer_size=32k
sort_buffer_size=32k
max_connections=16000
back_log=5000
innodb_open_files=4000
 
#Monitoring
performance-schema=0
 
#Percona Server specific
userstat=0
thread-statistics=0
 
#InnoDB General
innodb_buffer_pool_load_at_startup=1
innodb_buffer_pool_dump_at_shutdown=1
innodb_numa_interleave=1
innodb_file_per_table=1
innodb_file_format=barracuda
innodb_flush_method=O_DIRECT_NO_FSYNC
innodb_doublewrite=1
innodb_support_xa=1
innodb_checksums=1
 
#Concurrency
innodb_thread_concurrency=144
innodb_page_cleaners=8
innodb_purge_threads=4
innodb_spin_wait_delay=12 Good value for RO is 6, for RW and RC is 192
innodb_log_file_size=8G
innodb_log_files_in_group=16
innodb_buffer_pool_size=128G
innodb_buffer_pool_instances=128 #to avoid wait on InnoDB Buffer Pool mutex
innodb_io_capacity=18000
innodb_io_capacity_max=36000
innodb_flush_log_at_timeout=0
innodb_flush_log_at_trx_commit=2
innodb_flush_sync=1
innodb_adaptive_flushing=1
innodb_flush_neighbors = 0
innodb_max_dirty_pages_pct=90
innodb_max_dirty_pages_pct_lwm=10
innodb_lru_scan_depth=4000
innodb_adaptive_hash_index=0
innodb_change_buffering=none #can be inserts, workload-specific
optimizer_switch="index_condition_pushdown=off" #workload-specific
```
> Situation Problem with 2 nodes
```
- [x] Shutdown through init or systemd
     --- don't worried after node become to online, it will be sync it'selft
- [x] crashes or suffers a loss of network connectivity
     [firt solution]
      --- log into the database client and run the following command:
      mysql> SET GLOBAL wsrep_provider_options='pc.bootstrap=YES';
     
     [second solution ]
      --- log into the database client and run the following command:
      mysql> SET GLOBAL wsrep_provider_options='pc.ignore_sb=TRUE';
      
      * Warning: dangerous in a multi-master setup
```
> Adding new node to Galera Cluster
```
(1) edit all nodes:
wsrep_cluster_address="gcomm://172.18.111.61,172.18.111.62,172.18.111.63,NEW_NODE"

(2) stop and restart all nodes
   * attended : new node may be can't not start because service timeout (default 90s), big db use huge time to syncing
   # nano /lib/systemd/system/mariadb.service
     added:
     
     [Service]
     # set Starting Timetout to infinity
     TimeoutStartSec = 0 
     
   # systemctl daemon-reload
```
