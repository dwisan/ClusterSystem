>GlusterFS : Distributed Replicated and Striped Glusterfs Volume Setting

![ScreenShot]()

```
# 2 Replicated volume, each have 2 brick striped
root@node01:~# mkdir -p /glusterfs/replica-strip
root@node02:~# mkdir -p /glusterfs/replica-strip 
root@node03:~# mkdir -p /glusterfs/replica-strip 
root@node04:~# mkdir -p /glusterfs/replica-strip

root@node01:~# gluster volume create vol_replica-strip replica 2 strip 2 transport tcp \
node01:/glusterfs/replica-strip \
node02:/glusterfs/replica-strip \
node03:/glusterfs/replica-strip \
node04:/glusterfs/replica-strip force

root@node01:~# gluster volume start vol_replica-strip

# Checking volume info
root@node01:~# gluster volume info vol_replica-strip

Volume Name: vol_replica-strip
Type: Striped-Replicate
Volume ID: 0a3458bc-a345-42a0-b57a-4c34a93cd4af
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/replica-strip
Brick2: node02:/glusterfs/replica-strip
Brick3: node03:/glusterfs/replica-strip
Brick4: node04:/glusterfs/replica-strip
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

# Checking volume status
root@node01:~# gluster volume status vol_replica-strip

Status of volume: vol_replica-strip
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/replica-strip       49157     0          Y       8588 
Brick node02:/glusterfs/replica-strip       49157     0          Y       8378 
Brick node03:/glusterfs/replica-strip       49157     0          Y       8315 
Brick node04:/glusterfs/replica-strip       49157     0          Y       8305 
Self-heal Daemon on localhost               N/A       N/A        Y       8614 
Self-heal Daemon on node03                  N/A       N/A        Y       8339 
Self-heal Daemon on node04                  N/A       N/A        Y       8329 
Self-heal Daemon on node02                  N/A       N/A        Y       8402 
 
Task Status of Volume vol_replica-strip
------------------------------------------------------------------------------
There are no active volume tasks
```
