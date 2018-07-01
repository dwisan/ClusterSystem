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
>การตั้งค่าระบบเครือข่ายของแต่ละโหนดที่ใช้งาน
```
root@node01:~# nano /etc/hostname
node01
root@node02:~# nano /etc/hostname
node02
root@node03:~# nano /etc/hostname
node03
root@node04:~# nano /etc/hostname
node04

#
#In Ubuntu 16.04 LTS
#

* * * Node01 * * *
root@node01:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.101
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@node01:~# reboot

* * * Node02 * * *
root@node02:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.102
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@node02:~# reboot

* * * Node03 * * *
root@node03:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.103
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@node03:~# reboot

* * * Node04 * * *
root@node04:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.104
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@node04:~# reboot

#
#In Ubuntu 18.04 LTS
#

* * * Node01 * * *
root@node01:~# nano /etc/netplan/01-netcfg.yaml

network:
 version: 2
 renderer: networkd
 ethernets:
  enp2s0:
    dhcp4: no
    dhcp6: no
    addresses: [172.18.111.101/24]
    gateway4: 172.18.111.1
    nameservers:
      addresses: [172.18.111.2,172.18.111.3]

root@node01:~# netplan apply

* * * Node02 * * *
root@node01:~# nano /etc/netplan/01-netcfg.yaml

network:
 version: 2
 renderer: networkd
 ethernets:
  enp2s0:
    dhcp4: no
    dhcp6: no
    addresses: [172.18.111.102/24]
    gateway4: 172.18.111.1
    nameservers:
      addresses: [172.18.111.2,172.18.111.3]

root@node02:~# netplan apply

* * * Node03 * * *
root@node03:~# nano /etc/netplan/01-netcfg.yaml

network:
 version: 2
 renderer: networkd
 ethernets:
  enp2s0:
    dhcp4: no
    dhcp6: no
    addresses: [172.18.111.103/24]
    gateway4: 172.18.111.1
    nameservers:
      addresses: [172.18.111.2,172.18.111.3]

root@node03:~# netplan apply

* * * Node04 * * *
root@node04:~# nano /etc/netplan/01-netcfg.yaml

network:
 version: 2
 renderer: networkd
 ethernets:
  enp2s0:
    dhcp4: no
    dhcp6: no
    addresses: [172.18.111.104/24]
    gateway4: 172.18.111.1
    nameservers:
      addresses: [172.18.111.2,172.18.111.3]

root@node01:~# netplan apply

```
>อัพเดทระบบปฏิบัติการและซอฟต์แวร์บนโหนดทุกโหนด
 ```
 #apt update && apt upgrade -y
 ```
>ระบุโหนด ทุกโหนดที่ใช้งาน ลงใน /etc/hosts 
  ```
  127.0.0.1       localhost
  127.0.1.1       nodeXX
  172.18.111.101 node01
  172.18.111.102 node02
  172.18.111.103 node03
  172.18.111.104 node04
  ```
>ติดตั้งซอฟต์แวร์ Gluster Server บนทุกโหนดที่ใช้งาน

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
peer probe: success. Probe on localhost not needed
root@node01:~# gluster peer probe node02
peer probe: success.
root@node01:~# gluster peer probe node03
peer probe: success.
root@node01:~# gluster peer probe node04
peer probe: success.
```
>Checking peer status
```
root@node01:~# gluster peer status

Hostname: node02
Uuid: 3cdaacc7-cbdb-4209-8ff2-cafe6e4536e3
State: Peer in Cluster (Connected)

Hostname: node03
Uuid: e30be633-376f-45e3-8eea-ab88acec98a4
State: Peer in Cluster (Connected)

Hostname: node04
Uuid: ea1d45e3-bd82-4ec4-b6fa-830640b8170b
State: Peer in Cluster (Connected)
```
>Checking pool list
```
root@node01:~# gluster pool list
UUID                                    Hostname        State
3cdaacc7-cbdb-4209-8ff2-cafe6e4536e3    node02          Connected 
e30be633-376f-45e3-8eea-ab88acec98a4    node03          Connected 
ea1d45e3-bd82-4ec4-b6fa-830640b8170b    node04          Connected 
e2e55fba-fc46-4ed1-9655-b1b1a0b3e439    localhost       Connected 

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
>GlusterFS : Striped Glusterfs Volume Setting
* Summary of Striped Replicated Volume
- [x] Striped volume does not provide redundancy
- [x] disaster in one brick can cause data loss
- [x] number of stripe must be equal to number of bricks
- [x] provides added performance if large number of clients are accessing the same volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412387/f411fa56-ef5f-11e4-8e78-a0896a47625a.png)
```
# Create brick Directory for Strip 
root@node01:~# mkdir -p /glusterfs/striped
root@node02:~# mkdir -p /glusterfs/striped 
root@node03:~# mkdir -p /glusterfs/striped 
root@node04:~# mkdir -p /glusterfs/striped 

# Create volume for Strip a file to 4 brick
root@node01:~# gluster volume create vol_striped stripe 4 transport tcp \
node01:/glusterfs/striped \
node02:/glusterfs/striped \
node03:/glusterfs/striped \
node04:/glusterfs/striped force

#Start Volume
root@node01:~# gluster volume start vol_striped 


# Checking volume info
root@node01:~# gluster volume info vol_striped 

Volume Name: vol_striped
Type: Stripe
Volume ID: 44c507d2-0501-4a46-a85b-931947b6059e
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 4 = 4
Transport-type: tcp
Bricks:
Brick1: node01:/glusterfs/striped
Brick2: node02:/glusterfs/striped
Brick3: node03:/glusterfs/striped
Brick4: node04:/glusterfs/striped
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# Checking volume status
root@node01:~# gluster volume status vol_striped

Status of volume: vol_striped
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick node01:/glusterfs/striped             49155     0          Y       8387 
Brick node02:/glusterfs/striped             49155     0          Y       8220 
Brick node03:/glusterfs/striped             49155     0          Y       8156 
Brick node04:/glusterfs/striped             49155     0          Y       8147 
 
Task Status of Volume vol_striped
------------------------------------------------------------------------------
There are no active volume tasks
```
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

