>MySQL Cluster
To implement a MySQL Cluster, we need 3 different types of nodes:
- [x] Mamagement Node -- Used for monitoring and configuring the cluster
- [x] Data Node -- used to store the data they provide automatic sharding and can handle replication
- [x] SQL Node -- MySQL Server interfaces for connecting to all nodes

>Downloading and Installing MySQL Cluster
# apt-get install libaio1
# wget http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.4/mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64.tar.gz
# tar -zxvf mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64.tar.gz
# mv mysql-cluster-gpl-7.4.11-linux-glibc2.5-x86_64 mysql/
# mv mysql /usr/local
