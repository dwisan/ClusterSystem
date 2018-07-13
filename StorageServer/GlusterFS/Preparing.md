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
172.18.111.101
root@node01:~# nano /etc/hosts
127.0.0.1       localhost
127.0.1.1       172-18-111-101

root@node02:~# nano /etc/hostname
172.18.111.102
root@node02:~# nano /etc/hostname
127.0.0.1       localhost
127.0.1.1       172-18-111-102

root@node03:~# nano /etc/hostname
172.18.111.103
root@node03:~# nano /etc/hostname
127.0.0.1       localhost
127.0.1.1       172-18-111-103

root@node04:~# nano /etc/hostname
172.18.111.104
root@node04:~# nano /etc/hostname
127.0.0.1       localhost
127.0.1.1       172-18-111-104
#
#In Ubuntu 16.04 LTS
#

* * * 172.18.111.101 * * *
root@172-18-111-101:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.101
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@172-18-111-101:~# reboot

* * * 172.18.111.102 * * *
root@172-18-111-102:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.102
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@172-18-111-102:~# reboot

* * * 172.18.111.103 * * *
root@172-18-111-103:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.103
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@172-18-111-103:~# reboot

* * * 172.18.111.104 * * *
root@172-18-111-104:~# nano /etc/network/interface

auto enp2s0 
iface enp2s0 inet static
        address 172.18.111.104
        netmask 255.255.255.0
        network 172.18.111.0
        gateway 172.18.111.1
        dns-nameservers 172.18.111.2,172.18.111.3

root@172-18-111-104:~# reboot

#
#In Ubuntu 18.04 LTS
#

* * * 172-18-111-101 * * *
root@172-18-111-101:~# nano /etc/netplan/01-netcfg.yaml

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

root@172-18-111-101:~# netplan apply

* * * 172-18-111-102 * * *
root@172-18-111-102:~# nano /etc/netplan/01-netcfg.yaml

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

root@172-18-111-102:~# netplan apply

* * * 172-18-111-103 * * *
root@172-18-111-103:~# nano /etc/netplan/01-netcfg.yaml

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

root@172-18-111-103:~# netplan apply

* * * 172-18-111-104 * * *
root@172-18-111-104:~# nano /etc/netplan/01-netcfg.yaml

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

root@172-18-111-104:~# netplan apply

```
>อัพเดทระบบปฏิบัติการและซอฟต์แวร์บนโหนดทุกโหนด
 ```
 #apt update && apt upgrade -y
 ```
>ติดตั้งซอฟต์แวร์ Gluster Server บนทุกโหนดที่ใช้งาน

```
root@172-18-111-xxx:~# apt-get install software-properties-common

#In Ubuntu 16.04 LTS
root@172-18-111-xxx:~# add-apt-repository ppa:gluster/glusterfs-4.1

#In Ubuntu 18.04 LTS
root@172-18-111-xxx:~# add-apt-repository ppa:gluster/glusterfs-4.1

root@172-18-111-xxx:~# apt-get update
root@172-18-111-xxx:~# apt-get -y install glusterfs-server
```
> edit binding adddress
```
root@172-18-111-xxx:~# nano /etc/glusterfs/glusterd.vol

volume management
    type mgmt/glusterd
    option working-directory /var/lib/glusterd
    option transport-type socket,rdma
    option transport.socket.bind-address 172.18.111.xxx <------ 
    option transport.socket.keepalive-time 10
    option transport.socket.keepalive-interval 2
    option transport.socket.read-fail-log off
    option ping-timeout 0
    option event-threads 1
#   option lock-timer 180
#   option transport.address-family inet6
#   option base-port 49152
#   option max-port  65535
end-volume

```
> start and enable start up
```
root@172-18-111-xxx:~# service glusterd start
root@172-18-111-xxx:~# systemctl enable glusterd
```
> Probe Nodes to Cluster, 
```
root@172.18.111.101:~# gluster peer probe 172.18.111.101
peer probe: success. Probe on localhost not needed
root@172.18.111.101:~# gluster peer probe 172.18.111.102
peer probe: success.
root@172.18.111.101:~# gluster peer probe 172.18.111.103
peer probe: success.
root@172.18.111.101:~# gluster peer probe 172.18.111.104
peer probe: success.
```
>Checking peer status
```
root@172.18.111.101:~# gluster peer status

Hostname: 172.18.111.102
Uuid: 3cdaacc7-cbdb-4209-8ff2-cafe6e4536e3
State: Peer in Cluster (Connected)

Hostname: 172.18.111.103
Uuid: e30be633-376f-45e3-8eea-ab88acec98a4
State: Peer in Cluster (Connected)

Hostname: 172.18.111.104
Uuid: ea1d45e3-bd82-4ec4-b6fa-830640b8170b
State: Peer in Cluster (Connected)
```
>Checking pool list
```
root@172.18.111.101:~# gluster pool list
UUID                                    Hostname        State
3cdaacc7-cbdb-4209-8ff2-cafe6e4536e3    172.18.111.102  Connected 
e30be633-376f-45e3-8eea-ab88acec98a4    172.18.111.103  Connected 
ea1d45e3-bd82-4ec4-b6fa-830640b8170b    172.18.111.104  Connected 
e2e55fba-fc46-4ed1-9655-b1b1a0b3e439    localhost       Connected 

```


