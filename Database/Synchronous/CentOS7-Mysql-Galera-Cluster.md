> Step 1: enable the Galera Cluster yum repositories
```
vi /etc/yum.repos.d/galera.repo

[galera]
name = Galera
baseurl = http://releases.galeracluster.com/galera-3/centos/7/x86_64/
gpgkey = http://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

[mysql-wsrep]
name = MySQL-wsrep
baseurl = http://releases.galeracluster.com/mysql-wsrep-5.7.21-25.14/centos/7/x86_64/
gpgkey = http://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1
```
>Step 2: Install Galera prerequisite packages
```
yum -y install galera-3 mysql-wsrep-5.7 rsync lsof policycoreutils-python firewalld
```
>Step 3:Enable Galera Service
```
systemctl enable mysqld
```
>Step 4: Enable Firewalld Service
```
systemctl enable firewalld
systemctl start firewalld
```
>Step 5: Configure Galera Firewalld exeptions
```
firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --zone=public --add-port=4444/tcp --permanent
firewall-cmd --zone=public --add-port=4567/tcp --permanent
firewall-cmd --zone=public --add-port=4567/udp --permanent
firewall-cmd --zone=public --add-port=4568/tcp --permanent
firewall-cmd --reload
```
>Step 6: (Optional) Configure SELinux
```
semanage port -a -t mysqld_port_t -p tcp 4567
semanage port -a -t mysqld_port_t -p udp 4567
semanage port -a -t mysqld_port_t -p tcp 4568
semanage port -a -t mysqld_port_t -p tcp 4444
semanage permissive -a mysqld_t
```

>Step 7: Edit /etc/my.cnf
```
cp /etc/my.cnf /etc/my.cnf.bak
vi /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
binlog_format=ROW
bind-address=0.0.0.0
default_storage_engine=innodb
innodb_autoinc_lock_mode=2
innodb_flush_log_at_trx_commit=0
innodb_buffer_pool_size=122M
wsrep_provider=/usr/lib64/galera-3/libgalera_smm.so
wsrep_provider_options="gcache.size=300M; gcache.page_size=300M"
wsrep_cluster_name="galera_cluster1"
wsrep_cluster_address="gcomm://10.1.0.11:3306,10.1.0.12:3306,10.1.0.13:3306"
wsrep_sst_method=rsync
server_id=1
wsrep_node_address="10.1.0.11"
wsrep_node_name="mysql01"
[mysql_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
>Step 8: Create MySQL log file
```
touch /var/log/mysqld.log
chown mysql:mysql /var/log/mysqld.log
```

>Step 9: Repeat steps in other node servers
```
server_id=1
wsrep_node_address="10.1.0.11"
wsrep_node_name="mysql01"
```
>Step 10: Start MySQL service
```
/usr/bin/mysqld_bootstrap
```
>Step 11: MySQL 5.7 Password configuration
```
grep 'temporary password' /var/log/mysqld.log
```
> Step 12: Secure MySQL
```
/usr/bin/mysql_secure_installation
```
> Step 13: Confirm Galera Replication
```

ref: https://www.dreamvps.com/tutorials/install-mysql-galera-cluster-centos-7/
```
