>GlusterFS : Replicated Glusterfs Volume Setting
* Summary of Replicated Volume are mentioned below
- [x] Useful where availability is the priority
- [x] Useful where redundancy is the priority
- [x] Number of bricks should be equal to number of replicas. 

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412379/d75272a6-ef5f-11e4-869a-c355e8505747.png)
```
# Create brick Directory for Replicated volume
root@172-18-111-101:~# mkdir -p /glusterfs/replica 
root@172-18-111-101:~# mkdir -p /glusterfs/replica 

# Create Replicated volume 
root@172-18-111-101:~# gluster volume create vol_replica replica 2 transport tcp \
172.18.111.101:/glusterfs/replica \
172.18.111.102:/glusterfs/replica force

# Start volume
root@172-18-111-101:~# gluster volume start vol_replica

# Checking volume info
root@172-18-111-101:~# gluster volume info vol_replica

Volume Name: vol_replica
Type: Replicate
Volume ID: 640fe8cc-8ac2-4e8c-b67d-af1cd83dc788
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 172.18.111.101:/glusterfs/replica
Brick2: 172.18.111.102:/glusterfs/replica
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off

# Checking volume status
root@172-18-111-101:~# gluster volume status vol_replica

Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick 172.18.111.101:/glusterfs/replica     49152     0          Y       18955
Brick 172.18.111.102:/glusterfs/replica     49152     0          Y       18936
Self-heal Daemon on localhost               N/A       N/A        Y       18978
Self-heal Daemon on 172.18.111.102          N/A       N/A        Y       18959
 
Task Status of Volume vol_replica
------------------------------------------------------------------------------
There are no active volume tasks
```
>Expanding a gluster volume
- [x] Preparing for new hosts (multiple of replicate size)
```Shell
root@172-18-111-105:~# nano /etc/hostname
172-18-111-105
root@172-18-111-105:~# nano /etc/hosts
127.0.0.1       localhost
127.0.1.1       172-18-111-105

```
- [x] Installing Gluster Server Software
```
  Read in Preparing.md
```
- [x] Create brick 
```Shell
root@172-18-111-105:~# mkdir -p /glusterfs/distributed
```
- [x] Probe new Node
```Shell
root@172-18-111-101:~# gluster peer probe 172.18.111.105
peer probe: success.
```
- [x] Checking new Node in Pool list
```Shell
root@172-18-111-101:~# gluster pool list

```
- [x] add brick with associated of volume type
```Shell
root@172-18-111-101:~# gluster volume add-brick vol_distributed 172.18.111.105:/glusterfs/distributed force
volume add-brick: success

root@172-18-111-101:~# gluster volume info vol_distributed

V
root@172-18-111-101:~# gluster volume rebalance vol_distributed start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status
                                  
waiting status to completed


root@172-18-111-101:~# gluster volume rebalance vol_distributed fix-layout start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status

waiting status to completed

```

>shrink a gluster volume
```Shell
# remnove brick with associated of volume type
root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed start

root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed status


waiting for completed status


root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed commit
root@172-18-111-101:~# gluster peer detach 172.18.111.105
root@172-18-111-101:~# gluster peer status


root@172-18-111-101:~# gluster pool list


root@172-18-111-101:~# gluster volume rebalance vol_distributed start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status


waiting for status show completed


root@172-18-111-101:~# gluster volume rebalance vol_distributed fix-layout start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status


waiting for completed status


```
>Replace faulty brick
```Shell
root@172-18-111-101:~# gluster peer probe 172.18.111.105
root@172-18-111-101:~# gluster volume add-brick vol_distributed 172.18.111.105:/glusterfs/distributed force
root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.104:/glusterfs/distributed start
root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.104:/glusterfs/distributed commit
root@172-18-111-101:~# gluster peer detach 172.18.111.104
```
>GlusterFS : Clients' Settings

- [x] GlusterFS Native
```Shell
root@client:~# apt install glusterfs-client attr
root@client:~# mkdir /glusterfs
root@client:~# mount -t glusterfs -o acl 172.18.111.101:/vol_distributed /glusterfs
```
- [x] NFS mount
```Shell
root@client:~# apt-get -y install nfs-common 
root@client:~# systemctl enable rpcbind 
root@client:~# service rpcbind start
root@client:~# mount -t nfs -o vers=3,mountproto=tcp 172.18.111.101:/vol_distributed /glusterfs
```
- [x] Automatically Mounting Volumes
```Shell
root@client:~# nano /etc/fstab
172.18.111.101:/vol_distributed /glusterfs glusterfs defaults,_netdev 0 0

```
- [x] Test io
```
root@client:~# sysbench --test=fileio --file-total-size=150G prepare
root@client:~# sysbench --test=fileio --file-total-size=150G --file-test-mode=rndrw \
--init-rng=on --max-time=300 --max-requests=0 run
   
root@client:~# sysbench --test=fileio --file-total-size=150G cleanup

```
