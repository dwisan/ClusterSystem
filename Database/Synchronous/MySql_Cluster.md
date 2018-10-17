>MySQL Cluster
To implement a MySQL Cluster, need 3 different types of nodes:
- [x] Mamagement Node -- Used for monitoring and configuring the cluster
- [x] Data Node -- used to store the data they provide automatic sharding and can handle replication
- [x] SQL Node -- MySQL Server interfaces for connecting to all nodes

> Implementation example
  - [x] 2 x Management Node
  - [x] 6 x Data Node
  - [x] 4 x SQL Node
  
>Step # 1.0 <br />
Downloading and Installing MySQL Cluster Software to "/usr/local/mysql" <br />
On Management Node, Data Node, SQL Node *
```bash
bash# apt-get install libaio1
bash# wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.6/mysql-cluster-gpl-7.6.6-linux-glibc2.12-x86_64.tar.gz
bash# tar -zxvf mysql-cluster-gpl-7.6.6-linux-glibc2.12-x86_64.tar.gz
bash# mv mysql-cluster-gpl-7.6.6-linux-glibc2.12-x86_64 /usr/local/mysql
```
>Step # 2.0 <br />
Configure Management Node (2 node)
```
bash# cd /usr/local/mysql
bash# cp bin/ndb_mgm* /usr/local/bin/
bash# chmod +x /usr/local/bin/ndb_mgm*
bash# mkdir -p /usr/local/mysql/mysql-cluster/
```
>Step # 2.1 Create config.ini (2 node)
```
bash# nano /usr/local/mysql/mysql-cluster/config.ini

[NDB_MGMD DEFAULT]
PortNumber=1186
DataDir=/usr/local/mysql/mysql-cluster/

[ndb_mgmd]
NodeId=1
hostname=172.18.111.221

[ndb_mgmd]
NodeId=2
hostname=172.18.111.222

[ndbd default]

NoOfReplicas=2
# Using 2 replicas is recommended to guarantee availability of data; 
# using only 1 replica does not provide any redundancy, which means 
# that the failure of a single data node causes the entire cluster to 
# shut down. We do not recommend using more than 2 replicas, since 2 is 
# sufficient to provide high availability, and we do not currently test 
# with greater values for this parameter.

DataDir=/usr/local/mysql/cluster-data
BackupDataDir=/usr/local/mysql/cluster-data/BACKUP

LockPagesInMainMemory=1
# On Linux and Solaris systems, setting this parameter locks data node
# processes into memory. Doing so prevents them from swapping to disk,
# which can severely degrade cluster performance.

DataMemory=1500M
IndexMemory=384M

MaxNoOfConcurrentOperations=200000
MaxNoOfLocalOperations=220000

MaxNoOfTables=1024
MaxNofOfOrderedIndexes=256
MaxNoOfAttributes=50000
MaxNoOfOrderedIndexes=10000
LcpScanProgressTimeout=300

SchedulerSpinTimer=400
SchedulerExecutionTimer=100
RealTimeScheduler=1
# Setting these parameters allows you to take advantage of real-time scheduling
# of NDBCLUSTER threads to get higher throughput.

TimeBetweenGlobalCheckpoints=1000
TimeBetweenEpochs=200
DiskCheckpointSpeed=10M
DiskCheckpointSpeedInRestart=100M
RedoBuffer=32M

# CompressedLCP=1
# CompressedBackup=1
# Enabling CompressedLCP and CompressedBackup causes, respectively, local
# checkpoint files and backup files to be compressed, which can result in a space
# savings of up to 50% over noncompressed LCPs and backups.

LockExecuteThreadToCPU=1
LockMaintThreadsToCPU=0
# On systems with multiple CPUs, these parameters can be used to lock NDBCLUSTER
# threads to specific CPUs

[ndbd]
NodeId=11
hostname=172.18.111.223
   
[ndbd]
NodeId=12
hostname=172.18.111.224
   
[ndbd]
NodeId=13
hostname=172.18.111.225

[ndbd]
NodeId=14
hostname=172.18.111.226

[ndbd]
NodeId=15
hostname=172.18.111.227

[ndbd]
NodeId=16
hostname=172.18.111.228

[mysqld]
NodeId=21
hostname=172.18.111.229
   
[mysqld]
NodeId=22
hostname=172.18.111.230

[mysqld]
NodeId=23
hostname=172.18.111.231
   
[mysqld]
NodeId=24
hostname=172.18.111.232

```
> Step # 2.2 Run management Node (2 node)
```Shell
First Starting: After change configuation file must be Initialize First
bash# /usr/local/mysql/bin/ndb_mgmd -f /usr/local/mysql/mysql-cluster/config.ini --initial

Next Starting:
bash# /usr/local/mysql/bin/ndb_mgmd -f /usr/local/mysql/mysql-cluster/config.ini
```
> Step # 2.3 Show management Connection
```
bash# ndb_mgm -e show

[ndbd(NDB)]     6 node(s)
id=11 (not connected, accepting connect from 172.18.111.223)
id=12 (not connected, accepting connect from 172.18.111.224)
id=13 (not connected, accepting connect from 172.18.111.225)
id=14 (not connected, accepting connect from 172.18.111.226)
id=15 (not connected, accepting connect from 172.18.111.227)
id=16 (not connected, accepting connect from 172.18.111.228)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @172.18.111.221 (mysql-5.7.22 ndb-7.6.6)
id=2    @172.18.111.222 (mysql-5.7.22 ndb-7.6.6)

[mysqld(API)]   4 node(s)
id=21 (not connected, accepting connect from 172.18.111.229)
id=22 (not connected, accepting connect from 172.18.111.230)
id=23 (not connected, accepting connect from 172.18.111.231)
id=24 (not connected, accepting connect from 172.18.111.232)

```
>Step # 3.0 <br />
Configure Data Node (6 node)
```
bash# mkdir /usr/local/mysql/cluster-data
bash# mkdir /usr/local/mysql/cluster-data/BACKUP
bash# groupadd mysql
bash# useradd mysql -s /sbin/nologin -g mysql
bash# chown -R root:root /usr/local/mysql
bash# chown -R mysql:mysql /usr/local/mysql/cluster-data
```
>Step # 3.1 Create /etc/my.cnf (4 node)
```
bash# nano /etc/my.cnf

[mysql_cluster]
ndb-connectstring=172.18.111.221,172.18.111.222
```
>Step # 3.2 Run data node (4 node)
```
Firt Starting: Start with Initialize
bash# /usr/local/mysql/bin/ndbd --initial

Next Starting:
bash# /usr/local/mysql/bin/ndbd
```
> Step # 4.0 <br />
Configure SQL Node (4 node)
```
bash# mkdir /usr/local/mysql/data
bash# groupadd mysql
bash# useradd mysql -s /sbin/nologin -g mysql
bash# chown -R root:root /usr/local/mysql
bash# chown -R mysql:mysql /usr/local/mysql/data
bash# touch /var/log/log-slow-queries.log
```
>Step # 4.1 Create /etc/my.cnf (4 node)
```
bash# nano /etc/my.cnf

[mysqld]
ndbcluster
ndb-connectstring=172.18.111.221,172.18.111.222
default_storage_engine=ndbcluster
max_connections = 20000
slow_query_log = 1
slow_query_log_file=/var/log/log-slow-queries.log
log_queries_not_using_indexes
long_query_time = 10
open_files_limit = 100000
```
>Step # 4.2 initialize database for mysql Service (4 node)
```
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data
```
>Step # 4.3 Start Mysql Service and set root passward (4 node)
```
/usr/local/mysql/support-files/mysql.server start
/usr/local/mysql/bin/mysqladmin -u root -p password 1qa2ws3ed
```
>Step # 5.0 <br />
>Show Cluster Connection on management node
```
bash# ndb_mgm -e show

[ndbd(NDB)]     6 node(s)
id=11   @172.18.111.223  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 0, *)
id=12   @172.18.111.224  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 0)
id=13   @172.18.111.225  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 1)
id=14   @172.18.111.226  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 1)
id=15   @172.18.111.227  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 2)
id=16   @172.18.111.228  (mysql-5.7.22 ndb-7.6.6, Nodegroup: 2)

[ndb_mgmd(MGM)] 2 node(s)
id=1    @172.18.111.221  (mysql-5.7.22 ndb-7.6.6)
id=2    @172.18.111.222  (mysql-5.7.22 ndb-7.6.6)

[mysqld(API)]   4 node(s)
id=21   @172.18.111.229  (mysql-5.7.22 ndb-7.6.6)
id=22   @172.18.111.230  (mysql-5.7.22 ndb-7.6.6)
id=23   @172.18.111.231  (mysql-5.7.22 ndb-7.6.6)
id=24   @172.18.111.232  (mysql-5.7.22 ndb-7.6.6)
```
> BenchMask With sysbench
- [x] Preparing Data
```
# sysbench --test=oltp --oltp-table-size=1000000 --db-driver=mysql --mysql-port=3306 --mysql-socket=/tmp/mysql.sock --mysql-table-engine=ndbcluster --mysql-ssl=no --mysql-db=dbtest --mysql-user=root --mysql-password=1qa2ws3ed prepare
```
- [x] Runing Test
```
# sysbench --test=oltp --oltp-table-size=1000000 --db-driver=mysql --mysql-port=3306 --mysql-socket=/tmp/mysql.sock --mysql-table-engine=ndbcluster --mysql-ssl=no --mysql-db=dbtest --mysql-user=root --mysql-password=1qa2ws3ed --max-time=60 --oltp-read-only=on --max-requests=0 --num-threads=4 run
```
- [x] The Result
```
OLTP test statistics:
    queries performed:
        read:                            122822
        write:                           0
        other:                           17546
        total:                           140368
    transactions:                        8773   (146.18 per sec.)
    deadlocks:                           0      (0.00 per sec.)
    read/write requests:                 122822 (2046.51 per sec.)
    other operations:                    17546  (292.36 per sec.)

Test execution summary:
    total time:                          60.0154s
    total number of events:              8773
    total time taken by event execution: 239.9738
    per-request statistics:
         min:                                 19.04ms
         avg:                                 27.35ms
         max:                                 38.57ms
         approx.  95 percentile:              30.66ms

Threads fairness:
    events (avg/stddev):           2193.2500/2.68
    execution time (avg/stddev):   59.9934/0.00
```
> Using HAProxy to Load Banlancer
```
# nano /etc/haproxy/haproxy.cfg

frontend mariadb
        bind 0.0.0.0:3306
        mode tcp
        default_backend mariadb_galera

backend mariadb_galera
        balance leastconn
        mode tcp
        option tcpka
        server mariadb1 172.18.111.229:3306 check weight 1
        server mariadb2 172.18.111.230:3306 check weight 1
        server mariadb3 172.18.111.231:3306 check weight 1
        server mariadb4 172.18.111.232:3306 check weight 1
```
