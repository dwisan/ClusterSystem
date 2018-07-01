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
