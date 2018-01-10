>MySQL Cluster
To implement a MySQL Cluster, we need 3 different types of nodes:
- [x] Mamagement Node -- Used for monitoring and configuring the cluster
- [x] Data Node -- used to store the data they provide automatic sharding and can handle replication
- [x] SQL Node -- MySQL Server interfaces for connecting to all nodes

>Downloading and Installing MySQL Cluster
```
apt-get install libaio1
wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.4/mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64.tar.gz
tar -zxvf mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64.tar.gz
mv mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64 mysql/
mv mysql /usr/local
```
>Configure Management Node
```
cd /usr/local/mysql
cp bin/ndb_mgm* /usr/local/bin/
chmod +x /usr/local/bin/ndb_mgm*
mkdir -p /usr/local/mysql/mysql-cluster/
nano /usr/local/mysql/mysql-cluster/config.ini

[TCP DEFAULT]
SendBufferMemory=2M
ReceiveBufferMemory=2M

[NDB_MGMD DEFAULT]
PortNumber=1186
DataDir=/usr/local/mysql/mysql-cluster/

[ndb_mgmd]
NodeId=1
hostname=172.18.111.101
LogDestination=FILE:filename=ndb_1_cluster.log,maxsize=10000000,maxfiles=6

[ndbd default]
NoOfReplicas=3
DataMemory=256M
IndexMemory=256M
LockPagesInMainMemory=1
DataDir=DataDir=/usr/local/mysql/cluster-data

MaxNoOfConcurrentOperations=1000
MaxNoOfConcurrentTransactions=1024

StringMemory=25
MaxNoOfTables=1024
MaxNoOfOrderedIndexes=256
MaxNoOfUniqueHashIndexes=64
MaxNoOfAttributes=2560
MaxNoOfTriggers=2560

FragmentLogFileSize=16M
InitFragmentLogFiles=FULL
NoOfFragmentLogFiles=5
RedoBuffer=8M

TimeBetweenGlobalCheckpoints=1000
DiskCheckpointSpeedInRestart=20M
DiskCheckpointSpeed=4M
TimeBetweenLocalCheckpoints=20
CompressedLCP=1

HeartbeatIntervalDbDb=15000
HeartbeatIntervalDbApi=15000

BackupMaxWriteSize=1M
BackupDataBufferSize=12M
BackupLogBufferSize=8M
BackupMemory=20M
CompressedBackup=1

SharedGlobalMemory=10M
DiskPageBufferMemory=32M

[ndbd]
NodeId=2
hostname=172.18.111.102
   
[ndbd]
NodeId=3
hostname=172.18.111.103
   
[ndbd]
NodeId=4
hostname=172.18.111.104

[ndbd]
NodeId=5
hostname=172.18.111.105
   
[mysqld]
NodeId=6
hostname=172.18.111.106
   
[mysqld]
NodeId=7
hostname=172.18.111.107

```
> Run management Node
```
/usr/local/mysql/bin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini --initial
```
> Show management Connection
```
ndb_mgm -e show

[ndbd(NDB)]     3 node(s)
id=2 (not connected, accepting connect from 172.18.111.102)
id=3 (not connected, accepting connect from 172.18.111.103)
id=4 (not connected, accepting connect from 172.18.111.104)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @172.18.111.101  (mysql-5.6.38 ndb-7.4.17)

[mysqld(API)]   3 node(s)
id=5 (not connected, accepting connect from 172.18.111.102)
id=6 (not connected, accepting connect from 172.18.111.103)
id=7 (not connected, accepting connect from 172.18.111.104)

```
>Configure Data Node
```
mkdir /usr/local/mysql/cluster-data
groupadd mysql
useradd mysql -s /sbin/nologin -g mysql
chown -R root:root /usr/local/mysql
chown -R mysql:mysql /usr/local/mysql/cluster-data
nano /etc/my.cnf

[mysql_cluster]
ndb-connectstring=172.18.111.101
```
> Run data node
```
/usr/local/mysql/bin/ndbd --initial
```
>Configure SQL Node
```
mkdir /usr/local/mysql/data
groupadd mysql
useradd mysql -s /sbin/nologin -g mysql
chown -R root:root /usr/local/mysql
chown -R mysql:mysql /usr/local/mysql/data
nano /etc/my.cnf

[mysqld]
ndbcluster
ndb-connectstring=172.18.111.101
default_storage_engine=ndbcluster

[mysql_cluster]
ndb-connectstring=172.18.111.101

/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data
/usr/local/mysql/support-files/mysql.server start
/usr/local/mysql/bin/mysqladmin -u root password 1qa2ws3ed
/usr/local/mysql/bin/mysql -u root -p
mysql> source /usr/local/mysql/share/ndb_dist_priv.sql
```
> Show Cluster Connection on management node
```
ndb_mgm -e show

[ndbd(NDB)]     3 node(s)
id=2    @172.18.111.102  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0, *)
id=3    @172.18.111.103  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0)
id=4    @172.18.111.104  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @172.18.111.101  (mysql-5.6.38 ndb-7.4.17)

[mysqld(API)]   3 node(s)
id=5    @172.18.111.102  (mysql-5.6.38 ndb-7.4.17)
id=6    @172.18.111.103  (mysql-5.6.38 ndb-7.4.17)
id=7    @172.18.111.104  (mysql-5.6.38 ndb-7.4.17)
```
> How to Create tables with ndbcluster
```
CREATE TABLE `mytest`.`tbls3` ( `aa` INT NOT NULL ,  `bb` INT NOT NULL ,  `cc` INT NOT NULL ,  `dd` INT NOT NULL ) ENGINE = ndbcluster;
```
> Check tables created on all sql node
