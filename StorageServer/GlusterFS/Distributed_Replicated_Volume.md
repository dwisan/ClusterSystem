>GlusterFS : Distributed + Replicated Glusterfs Volume
* Summary of Distributed Replicated Volume
- [x] Data is distributed across replicated sets
- [x] Good redundancy and good scaling

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412402/23a17eae-ef60-11e4-8813-a40a2384c5c2.png)
```
# Create brick Directory for Distributed + Replicated volume
root@node01:~# mkdir -p /glusterfs/dist-replica
root@node02:~# mkdir -p /glusterfs/dist-replica
root@node03:~# mkdir -p /glusterfs/dist-replica
root@node04:~# mkdir -p /glusterfs/dist-replica

# Create Distributed + Replicated volume 
# [2 bricks in a Replicated group and Distributed belong to Replicated groups]
root@node01:~# gluster volume create vol_dist-replica replica 2 transport tcp \
node01:/glusterfs/dist-replica \
node02:/glusterfs/dist-replica \
node03:/glusterfs/dist-replica \
node04:/glusterfs/dist-replica force

# Start volume
root@node01:~# gluster volume start vol_dist-replica

# Checking volume info
root@node01:~# gluster volume info vol_dist-replica

Volume Name: vol_dist-replica
Type: Distributed-Replicate
Volume ID: 6d732794-2cc8-4b04-8f97-4d7d48afe2d7
Status: Started
Snapshot Count: 0
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/dist-replica
Brick2: node02:/glusterfs/dist-replica
Brick3: node03:/glusterfs/dist-replica
Brick4: node04:/glusterfs/dist-replica
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

# Checking volume status
root@node01:~# gluster volume status vol_dist-replica

Status of volume: vol_dist-replica
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/dist-replica        49154     0          Y       8196 
Brick node02:/glusterfs/dist-replica        49154     0          Y       8056 
Brick node03:/glusterfs/dist-replica        49154     0          Y       7995 
Brick node04:/glusterfs/dist-replica        49154     0          Y       7984 
Self-heal Daemon on localhost               N/A       N/A        Y       8219 
Self-heal Daemon on node03                  N/A       N/A        Y       8018 
Self-heal Daemon on node02                  N/A       N/A        Y       8079 
Self-heal Daemon on node04                  N/A       N/A        Y       8007 
 
Task Status of Volume vol_dist-replica
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
