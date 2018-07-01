>GlusterFS : Distributed Striped Glusterfs Volume Setting
* Summary of Striped Replicated Volume
- [x] It stripes files across multiple bricks
- [x] Good for performance in accessing very large files
- [x] Brick count must always be in multiple of stripe count

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412394/0ce267d2-ef60-11e4-9959-43465a2a25f7.png)
```
# 2 Distributed volume, each have 2 brick striped 
root@node01:~# mkdir -p /glusterfs/diststriped 
root@node02:~# mkdir -p /glusterfs/diststriped 
root@node03:~# mkdir -p /glusterfs/diststriped 
root@node04:~# mkdir -p /glusterfs/diststriped 

root@node01:~# gluster volume create vol_diststriped stripe 2 transport tcp \
node01:/glusterfs/diststriped  \
node02:/glusterfs/diststriped  \
node03:/glusterfs/diststriped  \
node04:/glusterfs/diststriped force 

root@node01:~# gluster volume start vol_diststriped 

# Checking volume info
root@node01:~# gluster volume info vol_diststriped 

Volume Name: vol_diststriped
Type: Distributed-Stripe
Volume ID: d7cc9f2b-3feb-45a6-92e0-87fb89ffd86e
Status: Started
Snapshot Count: 0
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/diststriped
Brick2: node02:/glusterfs/diststriped
Brick3: node03:/glusterfs/diststriped
Brick4: node04:/glusterfs/diststriped
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# Checking volume status
root@node01:~# gluster volume status vol_diststriped

Status of volume: vol_diststriped
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/diststriped         49156     0          Y       8484 
Brick node02:/glusterfs/diststriped         49156     0          Y       8295 
Brick node03:/glusterfs/diststriped         49156     0          Y       8231 
Brick node04:/glusterfs/diststriped         49156     0          Y       8221 
 
Task Status of Volume vol_diststriped
------------------------------------------------------------------------------
There are no active volume tasks
```
