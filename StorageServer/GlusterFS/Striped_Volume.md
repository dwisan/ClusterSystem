>GlusterFS : Striped Glusterfs Volume Setting
* Summary of Striped Replicated Volume
- [x] Striped volume does not provide redundancy
- [x] disaster in one brick can cause data loss
- [x] number of stripe must be equal to number of bricks
- [x] provides added performance if large number of clients are accessing the same volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412387/f411fa56-ef5f-11e4-8e78-a0896a47625a.png)
```
# Create brick Directory for Strip 
root@node01:~# mkdir -p /glusterfs/striped
root@node02:~# mkdir -p /glusterfs/striped 
root@node03:~# mkdir -p /glusterfs/striped 
root@node04:~# mkdir -p /glusterfs/striped 

# Create volume for Strip a file to 4 brick
root@node01:~# gluster volume create vol_striped stripe 4 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped \
node03:/glusterfs/striped \
node04:/glusterfs/striped force

#Start Volume
root@node01:~# gluster volume start vol_striped 


# Checking volume info
root@node01:~# gluster volume info vol_striped 

Volume Name: vol_striped
Type: Stripe
Volume ID: 44c507d2-0501-4a46-a85b-931947b6059e
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 4 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/striped
Brick2: node02:/glusterfs/striped
Brick3: node03:/glusterfs/striped
Brick4: node04:/glusterfs/striped
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# Checking volume status
root@node01:~# gluster volume status vol_striped

Status of volume: vol_striped
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/striped             49155     0          Y       8387 
Brick node02:/glusterfs/striped             49155     0          Y       8220 
Brick node03:/glusterfs/striped             49155     0          Y       8156 
Brick node04:/glusterfs/striped             49155     0          Y       8147 
 
Task Status of Volume vol_striped
------------------------------------------------------------------------------
There are no active volume tasks
```
>Expanding a gluster volume
```
# add brick with associated of volume type
gluster volume add-brick {volume} {server:/brick}
gluster volume rebalance {volume} start
gluster volume rebalance {volume} fix-layout start
gluster volume rebalence {volume} migrate data start
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
