>MySQL Cluster
To implement a MySQL Cluster, need 3 different types of nodes:
- [x] Mamagement Node -- Used for monitoring and configuring the cluster
- [x] Data Node -- used to store the data they provide automatic sharding and can handle replication
- [x] SQL Node -- MySQL Server interfaces for connecting to all nodes

> Implementation example
  - [x] 1 x Management Node
  - [x] 4 x Data Node
  - [x] 4 x SQL Node
  - [x] Data Node and SQL Node, they are running on same machine.
  
>Step # 1.0 <br />
Downloading and Installing MySQL Cluster Software to "/usr/local/mysql" <br />
On Management Node, Data Node, SQL Node *
```bash
bash# apt-get install libaio1
bash# wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.6/mysql-cluster-gpl-7.6.3-linux-glibc2.12-x86_64.tar.gz
bash# tar -zxvf mysql-cluster-gpl-7.6.3-linux-glibc2.12-x86_64.tar.gz
bash# mv mysql-cluster-gpl-7.6.3-linux-glibc2.12-x86_64 /usr/local/mysql
```
>Step # 2.0 <br />
Configure Management Node
```
bash# cd /usr/local/mysql
bash# cp bin/ndb_mgm* /usr/local/bin/
bash# chmod +x /usr/local/bin/ndb_mgm*
bash# mkdir -p /usr/local/mysql/mysql-cluster/
```
>Step # 2.1 Create config.ini
```
bash# nano /usr/local/mysql/mysql-cluster/config.ini

[NDB_MGMD DEFAULT]
PortNumber=1186
DataDir=/usr/local/mysql/mysql-cluster/

[ndb_mgmd]
NodeId=1
hostname=172.18.111.100

[ndbd default]
NoOfReplicas=4
DataDir=/usr/local/mysql/cluster-data

[ndbd]
NodeId=2
hostname=172.18.111.101
   
[ndbd]
NodeId=3
hostname=172.18.111.102
   
[ndbd]
NodeId=4
hostname=172.18.111.103

[ndbd]
NodeId=5
hostname=172.18.111.104
   
[mysqld]
NodeId=6
hostname=172.18.111.101
   
[mysqld]
NodeId=7
hostname=172.18.111.102

[mysqld]
NodeId=8
hostname=172.18.111.103
   
[mysqld]
NodeId=9
hostname=172.18.111.104

```
> Step # 2.2 Run management Node
```Shell
bash# /usr/local/mysql/bin/ndb_mgmd -f /usr/local/mysql/mysql-cluster/config.ini --initial
```
> Step # 2.3 Show management Connection
```
bash# ndb_mgm -e show

[ndbd(NDB)]     4 node(s)
id=2 (not connected, accepting connect from 172.18.111.101)
id=3 (not connected, accepting connect from 172.18.111.102)
id=4 (not connected, accepting connect from 172.18.111.103)
id=5 (not connected, accepting connect from 172.18.111.104)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @172.18.111.101  (mysql-5.6.38 ndb-7.4.17)

[mysqld(API)]   4 node(s)
id=6 (not connected, accepting connect from 172.18.111.101)
id=7 (not connected, accepting connect from 172.18.111.102)
id=8 (not connected, accepting connect from 172.18.111.103)
id=9 (not connected, accepting connect from 172.18.111.104)

```
>Step # 3.0 <br />
Configure Data Node
```
bash# mkdir /usr/local/mysql/cluster-data
bash# groupadd mysql
bash# useradd mysql -s /sbin/nologin -g mysql
bash# chown -R root:root /usr/local/mysql
bash# chown -R mysql:mysql /usr/local/mysql/cluster-data
```
>Step # 3.1 Create /etc/my.cnf
```
bash# nano /etc/my.cnf

[mysql_cluster]
ndb-connectstring=172.18.111.101
```
>Step # 3.2 Run data node
```
bash# /usr/local/mysql/bin/ndbd --initial
```
> Step # 4.0 <br />
Configure SQL Node
```
bash# mkdir /usr/local/mysql/data
bash# groupadd mysql
bash# useradd mysql -s /sbin/nologin -g mysql
bash# chown -R root:root /usr/local/mysql
bash# chown -R mysql:mysql /usr/local/mysql/data
```
>Step # 4.1 Create /etc/my.cnf
```
bash# nano /etc/my.cnf

[mysqld]
ndbcluster
ndb-connectstring=172.18.111.100
default_storage_engine=ndbcluster

[mysql_cluster]
ndb-connectstring=172.18.111.101

```
>Step # 4.2 initialize database for mysql Service
```
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data
```
>Step # 4.3 Start Mysql Service and set root passward
```
/usr/local/mysql/support-files/mysql.server start
/usr/local/mysql/bin/mysqladmin -u root -p password 1qa2ws3ed
```
>Step # 5.0 <br />
>Show Cluster Connection on management node
```
bash# ndb_mgm -e show

[ndbd(NDB)]     4 node(s)
id=2    @172.18.111.101  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0, *)
id=3    @172.18.111.102  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0)
id=4    @172.18.111.103  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0)
id=5    @172.18.111.104  (mysql-5.6.38 ndb-7.4.17, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @172.18.111.100  (mysql-5.6.38 ndb-7.4.17)

[mysqld(API)]   4 node(s)
id=6    @172.18.111.101  (mysql-5.6.38 ndb-7.4.17)
id=7    @172.18.111.102  (mysql-5.6.38 ndb-7.4.17)
id=8    @172.18.111.103  (mysql-5.6.38 ndb-7.4.17)
id=9    @172.18.111.104  (mysql-5.6.38 ndb-7.4.17)
```
