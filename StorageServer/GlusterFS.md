> Networking Setting
  ##### Add hosts to /etc/hosts on all Nodes
  ```
  192.168.0.1 node01
  192.168.0.2 node02
  192.168.0.3 node03
  192.168.0.4 node04
  ```
>Install GlusterFS Server on all Nodes. 

```
root@nodeX:~# apt-get install software-properties-common
root@nodeX:~# add-apt-repository ppa:gluster/glusterfs-3.8
root@nodeX:~# apt-get update
root@nodeX:~# apt-get -y install glusterfs-server
root@nodeX:~# service glusterfs-server start
root@nodeX:~# systemctl enable glusterfs-server 
```
> Probe Nodes to Cluster
```
root@node01:~# gluster peer probe node01
root@node01:~# gluster peer probe node02
root@node01:~# gluster peer probe node03
root@node01:~# gluster peer probe node04
```
>GlusterFS : Distributed Glusterfs Volume Setting
```
# 4 Brick distributed
root@node01:~# mkdir /glusterfs/distributed
root@node02:~# mkdir /glusterfs/distributed
root@node03:~# mkdir /glusterfs/distributed
root@node04:~# mkdir /glusterfs/distributed

root@node01:~# gluster volume create vol_distributed transport tcp \
node01:/glusterfs/distributed \
node02:/glusterfs/distributed \
node03:/glusterfs/distributed \
node04:/glusterfs/distributed 

root@node01:~# gluster volume start vol_distributed 
```
>GlusterFS : Replicated Glusterfs Volume Setting
```
# 4 Brick replicated
root@node01:~# mkdir /glusterfs/replica 
root@node02:~# mkdir /glusterfs/replica 
root@node03:~# mkdir /glusterfs/replica 
root@node04:~# mkdir /glusterfs/replica 

root@node01:~# gluster volume create vol_replica replica 4 transport tcp \
node01:/glusterfs/replica \
node02:/glusterfs/replica \
node03:/glusterfs/replica \
node04:/glusterfs/replica 

root@node01:~# gluster volume start vol_replica 
```
>GlusterFS : Distributed + Replicated Glusterfs Volume
```
# 2 Distributed volume, each have 2 brick replicated 
root@node01:~# mkdir /glusterfs/dist-replica
root@node02:~# mkdir /glusterfs/dist-replica
root@node03:~# mkdir /glusterfs/dist-replica
root@node04:~# mkdir /glusterfs/dist-replica

root@node01:~# gluster volume create vol_dist-replica replica 2 transport tcp \
node01:/glusterfs/dist-replica \
node02:/glusterfs/dist-replica \
node03:/glusterfs/dist-replica \
node04:/glusterfs/dist-replica 

root@node01:~# gluster volume start vol_dist-replica 
```
>GlusterFS : Striped Glusterfs Volume Setting
```
# Strip a file to 4 brick
root@node01:~# mkdir /glusterfs/striped
root@node02:~# mkdir /glusterfs/striped 
root@node03:~# mkdir /glusterfs/striped 
root@node04:~# mkdir /glusterfs/striped 

root@node01:~# gluster volume create vol_striped stripe 4 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped \
node03:/glusterfs/striped \
node04:/glusterfs/striped 

root@node01:~# gluster volume start vol_striped 
```
>GlusterFS : Distributed Striped Glusterfs Volume Setting
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