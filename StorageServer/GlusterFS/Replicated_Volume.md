>GlusterFS : Replicated Glusterfs Volume Setting
* Summary of Replicated Volume are mentioned below
- [x] Useful where availability is the priority
- [x] Useful where redundancy is the priority
- [x] Number of bricks should be equal to number of replicas. 

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412379/d75272a6-ef5f-11e4-869a-c355e8505747.png)
```
# Create brick Directory for Replicated volume
root@node01:~# mkdir -p /glusterfs/replica 
root@node02:~# mkdir -p /glusterfs/replica 
root@node03:~# mkdir -p /glusterfs/replica 
root@node04:~# mkdir -p /glusterfs/replica 

# Create Replicated volume 
root@node01:~# gluster volume create vol_replica replica 4 transport tcp \
node01:/glusterfs/replica \
node02:/glusterfs/replica \
node03:/glusterfs/replica \
node04:/glusterfs/replica force

# Start volume
root@node01:~# gluster volume start vol_replica

# Checking volume info
root@node01:~# gluster volume info vol_replica

Volume Name: vol_replica
Type: Replicate
Volume ID: 47c18f5b-dc69-4e7d-8beb-5494decf025c
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 4 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/replica
Brick2: node02:/glusterfs/replica
Brick3: node03:/glusterfs/replica
Brick4: node04:/glusterfs/replica
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

# Checking volume status
root@node01:~# gluster volume status vol_replica

Status of volume: vol_replica
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/replica             49153     0          Y       8065 
Brick node02:/glusterfs/replica             49153     0          Y       7950 
Brick node03:/glusterfs/replica             49153     0          Y       7890 
Brick node04:/glusterfs/replica             49153     0          Y       7876 
Self-heal Daemon on localhost               N/A       N/A        Y       8088 
Self-heal Daemon on node02                  N/A       N/A        Y       7974 
Self-heal Daemon on node03                  N/A       N/A        Y       7914 
Self-heal Daemon on node04                  N/A       N/A        Y       7900 
 
Task Status of Volume vol_replica
------------------------------------------------------------------------------
There are no active volume tasks
```
