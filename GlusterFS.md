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
