```
  GlusterFS is a distributed scalable network-Attached storage filesystem that allows rapid provisioning 
of additional storage base on storage comsumption need.It has found applications including cloud computing,
streaming media services and content delivery networks.

Architecture
  * The GlusterFS architecture aggregates computer, storage, I/O resources into global namespace. 
  * Capacity is scaled by adding additional nodes or adding additional storage to each node.
  * Performance is increased by deploying storage among more nodes.
  * High availability is achieved by replacing data more way between nodes.
  
Terminology
 * Brick is th basic unit of storage, represented by directory on server disk.
 * Volume is a logical collection of Brick.
```
>Network Setting
  ##### add hosts to /etc/hosts with all Nodes
  ```
  172.18.111.101 node01
  172.18.111.102 node02
  172.18.111.103 node03
  172.18.111.104 node04
  ```
>Install GlusterFS Server

```

root@nodeX:~# apt-get install software-properties-common

#In Ubuntu 16.04 LTS
root@nodeX:~# add-apt-repository ppa:gluster/glusterfs-3.13

#In Ubuntu 18.04 LTS
root@nodeX:~# add-apt-repository ppa:gluster/glusterfs-4.1

root@nodeX:~# apt-get update
root@nodeX:~# apt-get -y install glusterfs-server

#In Ubuntu 16.04 LTS
root@nodeX:~# service glusterfs-server start
root@nodeX:~# systemctl enable glusterfs-server

#In Ubuntu 18.04 LTS
root@nodeX:~# service glusterd start
root@nodeX:~# systemctl enable glusterd
```
> Probe Nodes to Cluster, 
```
root@node01:~# gluster peer probe node01
root@node01:~# gluster peer probe node02
root@node01:~# gluster peer probe node03
root@node01:~# gluster peer probe node04
```
>GlusterFS : Distributed Glusterfs Volume Setting
* Summary of distributed Volume are mentioned below
- [x] Files are randomly distributed over the bricks in distributed volume
- [x] There is no redundancy
- [x] Data loss protection is provided by the underlying hardware (no protection from gluster )
- [x] Best for scaling size of the volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412364/ac0a300c-ef5f-11e4-8599-e7d06de1165c.png)
```
# Create brick Directory for distributed volume
root@node01:~# mkdir /glusterfs/distributed
root@node02:~# mkdir /glusterfs/distributed
root@node03:~# mkdir /glusterfs/distributed
root@node04:~# mkdir /glusterfs/distributed

# Create distributed volume 
root@node01:~# gluster volume create vol_distributed transport tcp \
node01:/glusterfs/distributed \
node02:/glusterfs/distributed \
node03:/glusterfs/distributed \
node04:/glusterfs/distributed force

# Start Volume
root@node01:~# gluster volume start vol_distributed 
```
>GlusterFS : Replicated Glusterfs Volume Setting
* Summary of Replicated Volume are mentioned below
- [x] Useful where availability is the priority
- [x] Useful where redundancy is the priority
- [x] Number of bricks should be equal to number of replicas. 

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412379/d75272a6-ef5f-11e4-869a-c355e8505747.png)
```
# Create brick Directory for Replicated volume
root@node01:~# mkdir /glusterfs/replica 
root@node02:~# mkdir /glusterfs/replica 
root@node03:~# mkdir /glusterfs/replica 
root@node04:~# mkdir /glusterfs/replica 

# Create Replicated volume 
root@node01:~# gluster volume create vol_replica replica 4 transport tcp \
node01:/glusterfs/replica \
node02:/glusterfs/replica \
node03:/glusterfs/replica \
node04:/glusterfs/replica force

# Start volume
root@node01:~# gluster volume start vol_replica 
```
>GlusterFS : Distributed + Replicated Glusterfs Volume
* Summary of Distributed Replicated Volume
- [x] Data is distributed across replicated sets
- [x] Good redundancy and good scaling

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412402/23a17eae-ef60-11e4-8813-a40a2384c5c2.png)
```
# Create brick Directory for Distributed + Replicated volume
root@node01:~# mkdir /glusterfs/dist-replica
root@node02:~# mkdir /glusterfs/dist-replica
root@node03:~# mkdir /glusterfs/dist-replica
root@node04:~# mkdir /glusterfs/dist-replica

# Create Distributed + Replicated volume 
# [2 bricks in a Replicated group and Distributed belong to Replicated groups]
root@node01:~# gluster volume create vol_dist-replica replica 2 transport tcp \
node01:/glusterfs/dist-replica \
node02:/glusterfs/dist-replica \
node03:/glusterfs/dist-replica \
node04:/glusterfs/dist-replica force

# Start volume
root@node01:~# gluster volume start vol_dist-replica 
```
>GlusterFS : Striped Glusterfs Volume Setting
* Summary of Striped Replicated Volume
- [x] Striped volume does not provide redundancy
- [x] disaster in one brick can cause data loss
- [x] number of stripe must be equal to number of bricks
- [x] provides added performance if large number of clients are accessing the same volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412387/f411fa56-ef5f-11e4-8e78-a0896a47625a.png)
```
# Create brick Directory for Strip 
root@node01:~# mkdir /glusterfs/striped
root@node02:~# mkdir /glusterfs/striped 
root@node03:~# mkdir /glusterfs/striped 
root@node04:~# mkdir /glusterfs/striped 

# Create volume for Strip a file to 4 brick
root@node01:~# gluster volume create vol_striped stripe 4 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped \
node03:/glusterfs/striped \
node04:/glusterfs/striped force

#Start Volume
root@node01:~# gluster volume start vol_striped 
```
>GlusterFS : Distributed Striped Glusterfs Volume Setting
* Summary of Striped Replicated Volume
- [x] It stripes files across multiple bricks
- [x] Good for performance in accessing very large files
- [x] Brick count must always be in multiple of stripe count

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412394/0ce267d2-ef60-11e4-9959-43465a2a25f7.png)
```
# 2 Distributed volume, each have 2 brick striped 
root@node01:~# mkdir /glusterfs/striped 
root@node02:~# mkdir /glusterfs/striped
root@node03:~# mkdir /glusterfs/striped
root@node04:~# mkdir /glusterfs/striped

root@node01:~# gluster volume create vol_striped stripe 2 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped \
node03:/glusterfs/striped \
node04:/glusterfs/striped 

root@node01:~# gluster volume start vol_striped 
```

>GlusterFS : Distributed Replicated and Striped Glusterfs Volume Setting

![ScreenShot]()

```
# 2 Replicated volume, each have 2 brick striped
root@node01:~# mkdir /glusterfs/replica-strip
root@node02:~# mkdir /glusterfs/replica-strip 
root@node03:~# mkdir /glusterfs/replica-strip 
root@node04:~# mkdir /glusterfs/replica-strip

root@node01:~# gluster volume create vol_replica-strip replica 2 strip 2 transport tcp \
node01:/glusterfs/replica-strip \
node02:/glusterfs/replica-strip \
node03:/glusterfs/replica-strip \
node04:/glusterfs/replica-strip 

root@node01:~# gluster volume start vol_replica-strip
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
