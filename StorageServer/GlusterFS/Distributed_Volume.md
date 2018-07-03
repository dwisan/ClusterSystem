>GlusterFS : Distributed Glusterfs Volume Setting
* Summary of distributed Volume are mentioned below
- [x] Files are randomly distributed over the bricks in distributed volume
- [x] There is no redundancy
- [x] Data loss protection is provided by the underlying hardware (no protection from gluster )
- [x] Best for scaling size of the volume

![ScreenShot](https://cloud.githubusercontent.com/assets/10970993/7412364/ac0a300c-ef5f-11e4-8599-e7d06de1165c.png)
```Shell
# Create brick Directory for distributed volume
root@172-18-111-101:~# mkdir -p /glusterfs/distributed
root@172-18-111-102:~# mkdir -p /glusterfs/distributed
root@172-18-111-103:~# mkdir -p /glusterfs/distributed
root@172-18-111-104:~# mkdir -p /glusterfs/distributed

# Create distributed volume 
root@172-18-111-101:~# gluster volume create vol_distributed transport tcp \
172.18.111.101:/glusterfs/distributed \
172.18.111.102:/glusterfs/distributed \
172.18.111.103:/glusterfs/distributed \
172.18.111.104:/glusterfs/distributed force

# Start Volume
root@172-18-111-101:~# gluster volume start vol_distributed

# Checking volume info
root@172-18-111-101:~# gluster volume info vol_distributed

Volume Name: vol_distributed
Type: Distribute
Volume ID: 36dd55c6-9929-4bdf-97c0-11f4ef14616c
Status: Started
Snapshot Count: 0
Number of Bricks: 4
Transport-type: tcp
Bricks:
Brick1: 172.18.111.101:/glusterfs/distributed
Brick2: 172.18.111.102:/glusterfs/distributed
Brick3: 172.18.111.103:/glusterfs/distributed
Brick4: 172.18.111.104:/glusterfs/distributed
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# Checking volume status
root@172-18-111-101:~# gluster volume status vol_distributed
Status of volume: vol_distributed
Gluster process                                    TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick 172.18.111.101:/glusterfs/distributed         49152     0          Y       7907 
Brick 172.18.111.102:/glusterfs/distributed         49152     0          Y       7781 
Brick 172.18.111.103:/glusterfs/distributed         49152     0          Y       7762 
Brick 172.18.111.104:/glusterfs/distributed         49152     0          Y       7748 
 
Task Status of Volume vol_distributed
------------------------------------------------------------------------------
There are no active volume tasks
```
>Expanding a gluster volume
- [x] Preparing for new hosts
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

UUID                                    Hostname        State
3cdaacc7-cbdb-4209-8ff2-cafe6e4536e3    172.18.111.102  Connected 
e30be633-376f-45e3-8eea-ab88acec98a4    172.18.111.103  Connected 
ea1d45e3-bd82-4ec4-b6fa-830640b8170b    172.18.111.104  Connected 
661ff4cb-de70-4dd6-8aa3-792842d24a5b    172.18.111.105  Connected <---
e2e55fba-fc46-4ed1-9655-b1b1a0b3e439    localhost       Connected 
```
- [x] add brick with associated of volume type
```Shell
root@172-18-111-101:~# gluster volume add-brick vol_distributed 172.18.111.105:/glusterfs/distributed force
volume add-brick: success

root@172-18-111-101:~# gluster volume info vol_distributed

Volume Name: vol_distributed
Type: Distribute
Volume ID: 36dd55c6-9929-4bdf-97c0-11f4ef14616c
Status: Started
Snapshot Count: 0
Number of Bricks: 5
Transport-type: tcp
Bricks:
Brick1: 172.18.111.101:/glusterfs/distributed
Brick2: 172.18.111.102:/glusterfs/distributed
Brick3: 172.18.111.103:/glusterfs/distributed
Brick4: 172.18.111.104:/glusterfs/distributed
Brick5: 172.18.111.105:/glusterfs/distributed <---
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

root@172-18-111-101:~# gluster volume rebalance vol_distributed start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status

Node       Rebalanced-files   size  scanned  failures   skipped    status    run time in h:m:s
---------    -----------   -------- -------- --------   -------  ----------   -------------
localhost      13624         7.3GB    34080      0        0     in progress     0:47:08
172.18.111.102 11370         6.1GB    30745      0        0     in progress     0:47:08
172.18.111.103 11968         6.5GB    31167      0        0     in progress     0:47:08
172.18.111.104  9478         5.0GB    30649      0        0     in progress     0:47:08
172.18.111.105     1       344.3KB     1117      0        1     in progress     0:47:08
                                  
waiting status to completed

Node       Rebalanced-files   size   scanned  failures   skipped   status     run time in h:m:s
---------    -----------   --------- -------- --------   -------- ---------     --------------
localhost       16468         8.8GB     41356      0         0       completed        0:53:47
172.18.111.102  12609         6.7GB     33734      0         0       completed        0:51:19
172.18.111.103  13218         7.1GB     34225      0         0       completed        0:51:15
172.18.111.104  10295         5.4GB     33632      0         0       completed        0:51:05
172.18.111.105      1       344.3KB      1160      0         1       completed        0:51:04

root@172-18-111-101:~# gluster volume rebalance vol_distributed fix-layout start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status

Node                                    status              run time in h:m:s
---------                             -----------                ------------
localhost                          fix-layout in progress      0:0:15
172.18.111.102                     fix-layout in progress      0:0:15
172.18.111.103                     fix-layout in progress      0:0:15
172.18.111.104                     fix-layout in progress      0:0:15
172.18.111.105                     fix-layout in progress      0:0:15

waiting status to completed

Node                                    status              run time in h:m:s
---------                            -----------            ------------
localhost                          fix-layout completed        0:5:0
172.18.111.102                     fix-layout completed        0:5:0
172.18.111.103                     fix-layout completed        0:5:0
172.18.111.104                     fix-layout completed        0:5:0
172.18.111.105                     fix-layout completed        0:4:59
```

>shrink a gluster volume
```Shell
# remnove brick with associated of volume type
root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed start

Running remove-brick with cluster.force-migration enabled can result in data corruption. 
It is safer to disable this option so that files that receive writes during migration are not migrated.
Files that are not migrated can then be manually copied after the remove-brick commit operation.
Do you want to continue with your current cluster.force-migration settings? (y/n) y
volume remove-brick start: success

root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed status

Node         Rebalanced-files    size       scanned      failures       skipped     status     run time in h:m:s
----            -----------     --------   -----------   ----------   -----------   ------        --------------
172.18.111.105      106        11.5MB         379            0             0      in progress        0:00:07

waiting for completed status

Node            Rebalanced-files    size     scanned    failures    skipped    status      run time in h:m:s
-----             -----------    ---------  ---------- ---------- ----------- -----------   --------------
172.18.111.105       60842        29.8GB      60982        0           0       completed        1:47:09

root@172-18-111-101:~# gluster volume remove-brick vol_distributed 172.18.111.105:/glusterfs/distributed commit
root@172-18-111-101:~# gluster peer detach 172.18.111.105
root@172-18-111-101:~# gluster peer status

Number of Peers: 3

Hostname: 172.18.111.103
Uuid: f0905a5b-257d-4459-86e5-5cd5083a28a9
State: Peer in Cluster (Connected)

Hostname: 172.18.111.104
Uuid: a96f98f9-0b35-4276-bb89-db8e51dbcacc
State: Peer in Cluster (Connected)

Hostname: 172.18.111.102
Uuid: ffe86122-6f35-4a8e-b7a1-c9f8be25df07
State: Peer in Cluster (Connected)

root@172-18-111-101:~# gluster pool list

UUID                                    Hostname        State
f0905a5b-257d-4459-86e5-5cd5083a28a9    172.18.111.103  Connected 
a96f98f9-0b35-4276-bb89-db8e51dbcacc    172.18.111.104  Connected 
ffe86122-6f35-4a8e-b7a1-c9f8be25df07    172.18.111.102  Connected 
d67693f6-6dfb-43e3-9f25-10d34c65412b    localhost       Connected 

root@172-18-111-101:~# gluster volume rebalance vol_distributed start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status

Node Rebalanced-files       size       scanned      failures       skipped       status     run time in h:m:s
---------      --------- ----------- -----------   -----------   -----------  ------------     --------------
localhost        1243       470.3MB     5248           0            54          in progress        0:05:15
172.18.111.103   1393       456.1MB     5480           0           229          in progress        0:05:15
172.18.111.104    916       422.0MB     5630           0             0          in progress        0:05:15
172.18.111.102   1198       519.1MB     4757  

waiting for status show completed

Node      Rebalanced-files   size   scanned   failures    skipped      status     run time in h:m:s
-----      -----------       ------ --------  -------   -----------  ----------    --------------
localhost         20389     10.1GB   79278       1           575       completed     1:53:18
172.18.111.103    19364      9.1GB   75054       1          1790       completed     1:53:25
172.18.111.104    17723      8.9GB   89660       0             0       completed     1:53:18
172.18.111.102    21785     10.7GB   80266       0          1506       completed     1:53:19

root@172-18-111-101:~# gluster volume rebalance vol_distributed fix-layout start
root@172-18-111-101:~# gluster volume rebalance vol_distributed status

Node                  status                run time in h:m:s
---------           -----------                ------------
localhost           fix-layout in progress        0:0:19
172.18.111.103      fix-layout in progress        0:0:19
172.18.111.104      fix-layout in progress        0:0:19
172.18.111.102      fix-layout in progress        0:0:19

waiting for completed status

Node                                    status              run time in h:m:s
---------                            -----------            ------------
localhost                          fix-layout completed        0:5:0
172.18.111.102                     fix-layout completed        0:5:0
172.18.111.103                     fix-layout completed        0:5:0
172.18.111.104                     fix-layout completed        0:5:0
172.18.111.105                     fix-layout completed        0:4:59
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

Extra file open flags: 0
128 files, 1.1719Gb each
150Gb total file size
Block size 16Kb
Number of random requests for random IO: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Threads started!
Time limit exceeded, exiting...
Done.

Operations performed:  6360 Read, 4240 Write, 13557 Other = 24157 Total
Read 99.375Mb  Written 66.25Mb  Total transferred 165.62Mb  (565.31Kb/sec)
   35.33 Requests/sec executed

Test execution summary:
    total time:                          300.0112s
    total number of events:              10600
    total time taken by event execution: 109.9779
    per-request statistics:
         min:                                  0.01ms
         avg:                                 10.38ms
         max:                                213.80ms
         approx.  95 percentile:              28.54ms

Threads fairness:
    events (avg/stddev):           10600.0000/0.00
    execution time (avg/stddev):   109.9779/0.00
```
