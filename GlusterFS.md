>Install GlusterFS Server on all Nodes in Cluster. 

```
root@node01:~# apt-get -y install glusterfs-server
root@node01:~# systemctl enable glusterfs-server 
```
>GlusterFS : Distributed Setting
```
root@node01:~# mkdir /glusterfs/distributed 
root@node01:~# gluster peer probe node02 
root@node01:~# gluster volume create vol_distributed transport tcp \
node01:/glusterfs/distributed \
node02:/glusterfs/distributed 
root@node01:~# gluster volume start vol_distributed 
```
>GlusterFS : Replication Setting
```
root@node01:~# mkdir /glusterfs/replica 
root@node01:~# gluster peer probe node02 
root@node01:~# gluster volume create vol_replica replica 2 transport tcp \
node01:/glusterfs/replica \
node02:/glusterfs/replica 
root@node01:~# gluster volume start vol_replica 
```
>GlusterFS : Striping Setting
```
root@node01:~# mkdir /glusterfs/striped 
root@node01:~# gluster peer probe node02 
root@node01:~# gluster volume create vol_striped stripe 2 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped 
root@node01:~# gluster volume start vol_striped 
```
>GlusterFS : Distributed + Replication
```
root@node01:~# mkdir /glusterfs/dist-replica

root@node01:~# gluster peer probe node02
root@node01:~# gluster peer probe node03
root@node01:~# gluster peer probe node04

root@node01:~# gluster volume create vol_dist-replica replica 2 transport tcp \
node01:/glusterfs/dist-replica \
node02:/glusterfs/dist-replica \
node03:/glusterfs/dist-replica \
node04:/glusterfs/dist-replica 

root@node01:~# gluster volume start vol_dist-replica 
```
>GlusterFS : Striping + Replication
```
root@node01:~# mkdir /glusterfs/strip-replica 
root@node01:~# gluster peer probe node02
root@node01:~# gluster peer probe node03
root@node01:~# gluster peer probe node04

root@node01:~# gluster volume create vol_strip-replica stripe 2 replica 2 transport tcp \
node01:/glusterfs/strip-replica \
node02:/glusterfs/strip-replica \
node03:/glusterfs/strip-replica \
node04:/glusterfs/strip-replica 

root@node01:~# gluster volume start vol_strip-replica 
```
