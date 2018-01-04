>Add a user for Ceph admin on all Nodes.
```
  root@host:~# adduser cephadm
  root@host:~# nano /etc/sudoers.d/ceph
    Defaults:cephadm !requiretty
    cephadm ALL = (root) NOPASSWD:ALL
  root@host:~# chmod 440 /etc/sudoers.d/ceph 
```
>Login as a Ceph admin user and configure Ceph. 
>Set SSH key-pair from Ceph Admin Node to all storage Nodes.
```
cephadm@host:~$ ssh-keygen
cephadm@host:~$ vi ~/.ssh/config

#
Host host
    Hostname host
    User cephadm
Host node01
    Hostname node01
    User cephadm
Host node02
    Hostname node02
    User cephadm
Host node03
    Hostname node03
    User cephadm
#

cephadm@host:~$ chmod 600 ~/.ssh/config
```
>transfer key file
```
cephadm@host:~$ ssh-copy-id node01 
cephadm@host:~$ ssh-copy-id node02 
cephadm@host:~$ ssh-copy-id node03 
```
>Install Ceph to all Nodes from Admin Node. 
```
cephadm@host:~$ sudo apt-get -y install ceph-deploy ceph-common ceph-mds
cephadm@host:~$ mkdir ceph
cephadm@host:~$ cd ceph
cephadm@host:~/ceph$ ceph-deploy new node01
cephadm@host:~/ceph$ ceph-deploy new node02
cephadm@host:~/ceph$ ceph-deploy new node03
cephadm@host:~/ceph$ vi ./ceph.conf
# add to the end
osd pool default size = 2

# install Ceph on each Node
cephadm@host:~/ceph$ ceph-deploy install dlp node01 node02 node03

# settings for monitoring and keys
cephadm@host:~/ceph$ ceph-deploy mon create-initial 
```
>Configure Ceph Cluster from Admin Node.
```
# prepare Object Storage Daemon
ubuntu@dlp:~/ceph$ ceph-deploy osd prepare node01:/storage01 node02:/storage02 node03:/storage03

# activate Object Storage Daemon
ubuntu@dlp:~/ceph$ ceph-deploy osd activate node01:/storage01 node02:/storage02 node03:/storage03

# transfer config files
ubuntu@dlp:~/ceph$ ceph-deploy admin dlp node01 node02 node03

# show status (display like follows if no ploblem)
ubuntu@dlp:~/ceph$ sudo ceph health
```
>clean settings and re-configure
```
# remove packages
ubuntu@dlp:~/ceph$ ceph-deploy purge dlp node01 node02 node03

# remove settings
ubuntu@dlp:~/ceph$ ceph-deploy purgedata dlp node01 node02 node03
ubuntu@dlp:~/ceph$ ceph-deploy forgetkeys
```
