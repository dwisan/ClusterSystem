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
DataDir=/usr/local/mysql/cluster-data
BackupDataDir=/usr/local/mysql/cluster-data/BACKUP

DataMemory=1500M
MaxNoOfConcurrentOperations=200000
MaxNoOfLocalOperations=220000

MaxNoOfTables=1024
MaxNoOfAttributes=50000
MaxNoOfOrderedIndexes=10000
LcpScanProgressTimeout=300

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
# sysbench --test=oltp --oltp-table-size=1000000 --db-driver=mysql --mysql-port=3306 --mysql-socket=/tmp/mysql.sock --mysql-table-engine=ndbcluster --mysql-db=dbtest --mysql-user=root --mysql-password=1qa2ws3ed prepare
```
- [x] Runing Test
```
# sysbench --test=oltp --oltp-table-size=1000000 --db-driver=mysql --mysql-port=3306 --mysql-socket=/tmp/mysql.sock --mysql-ssl=no --mysql-db=dbtest --mysql-user=root --mysql-password=1qa2ws3ed --max-time=60 --oltp-read-only=on --max-requests=0 --num-threads=4 run
```
- [x] The Result
```
OLTP test statistics:
    queries performed:
        read:                            2474822
        write:                           0
        other:                           353546
        total:                           2828368
    transactions:                        176773 (2946.16 per sec.)
    deadlocks:                           0      (0.00 per sec.)
    read/write requests:                 2474822 (41246.17 per sec.)
    other operations:                    353546 (5892.31 per sec.)

Test execution summary:
    total time:                          60.0013s
    total number of events:              176773
    total time taken by event execution: 239.5257
    per-request statistics:
         min:                                  0.82ms
         avg:                                  1.35ms
         max:                                  4.39ms
         approx.  95 percentile:               1.78ms

Threads fairness:
    events (avg/stddev):           44193.2500/378.27
    execution time (avg/stddev):   59.8814/0.00
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
