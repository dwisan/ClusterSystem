>GlusterFS : Distributed Glusterfs Volume Setting
* Summary of distributed Volume are mentioned below
- [x] Files are randomly distributed over the bricks in distributed volume
- [x] There is no redundancy
- [x] Data loss protection is provided by the underlying hardware (no protection from gluster )
- [x] Best for scaling size of the volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412364/ac0a300c-ef5f-11e4-8599-e7d06de1165c.png)
```
# Create brick Directory for distributed volume
root@node01:~# mkdir -p /glusterfs/distributed
root@node02:~# mkdir -p /glusterfs/distributed
root@node03:~# mkdir -p /glusterfs/distributed
root@node04:~# mkdir -p /glusterfs/distributed

# Create distributed volume 
root@node01:~# gluster volume create vol_distributed transport tcp \
node01:/glusterfs/distributed \
node02:/glusterfs/distributed \
node03:/glusterfs/distributed \
node04:/glusterfs/distributed force

# Start Volume
root@node01:~# gluster volume start vol_distributed

# Checking volume info
root@node01:~# gluster volume info vol_distributed

Volume Name: vol_distributed
Type: Distribute
Volume ID: 36dd55c6-9929-4bdf-97c0-11f4ef14616c
Status: Started
Snapshot Count: 0
Number of Bricks: 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/distributed
Brick2: node02:/glusterfs/distributed
Brick3: node03:/glusterfs/distributed
Brick4: node04:/glusterfs/distributed
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# Checking volume status
root@node01:~# gluster volume status vol_distributed
Status of volume: vol_distributed
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/distributed         49152     0          Y       7907 
Brick node02:/glusterfs/distributed         49152     0          Y       7781 
Brick node03:/glusterfs/distributed         49152     0          Y       7762 
Brick node04:/glusterfs/distributed         49152     0          Y       7748 
 
Task Status of Volume vol_distributed
------------------------------------------------------------------------------
There are no active volume tasks
```
>Expanding a gluster volume
- [x] Preparing for new hosts
```
root@node05:~# nano /etc/hostname
node05

root@node05:~# nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       node05

172.18.111.101  node01
172.18.111.102  node02
172.18.111.103  node03
172.18.111.104  node04
172.18.111.105  node05

root@node01-root@node04~# nano /etc/hosts
....
172.18.111.105  node05

```
- [x] Probe new Node
```
root@node01:~# gluster peer probe node05
peer probe: success.
```
- [x] add brick with associated of volume type
```
root@node01:~# gluster volume add-brick vol_distributed node01:/glusterfs/distributed
root@node01:~# gluster volume rebalance {volume} start
root@node01:~# gluster volume rebalance {volume} fix-layout start
root@node01:~# gluster volume rebalence {volume} migrate data start
```
>shrink a gluster volume
```
# remnove brick with associated of volume type
gluster volume remove-brick {volume} {server:/brick}
gluster volume rebalance {volume} start
gluster volume rebalance {volume} fix-layout start
gluster volume rebalence {volume} migrate data start
```

>GlusterFS : Clients' Settings

- [x] GlusterFS Native
```
root@client:~# apt install glusterfs-client attr
root@client:~# mkdir /glusterfs
root@client:~# mount -t glusterfs node01:/vol_strip-replica /glusterfs
```
- [x] NFS mount
```
root@client:~# apt-get -y install nfs-common 
root@client:~# systemctl enable rpcbind 
root@client:~# service rpcbind start
root@client:~# mount -t nfs -o vers=3,mountproto=tcp node01:/vol_strip-replica /glusterfs
```
- [x] Automatically Mounting Volumes
```
root@client:~# nano /etc/fstab

node01:/cluster1_volume /glusterfs glusterfs defaults,_netdev 0 0

```
