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
```
> edit binding adddress
```
root@nodeX:~# nano /etc/glusterfs/glusterd.vol

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


