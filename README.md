# TP1 : (re)Familiaration avec un système GNU/Linux

## 0. Préparation de la machine

### Setup de deux machines Rocky Linux configurées de façon basique.

**un accès internet (via la carte NAT)**

```
[leo@node1 ~]$ ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:66:e2:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85588sec preferred_lft 85588sec
    inet6 fe80::a00:27ff:fe66:e2f7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[...]
```

```
[leo@node2 ~]$ ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:dc:07:93 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85611sec preferred_lft 85611sec
    inet6 fe80::a00:27ff:fedc:793/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[...]
```

**un accès à un réseau local**

```
[leo@node1 ~]$ ping node2
PING node2 (10.101.1.12) 56(84) bytes of data.
64 bytes from node2 (10.101.1.12): icmp_seq=1 ttl=64 time=0.603 ms
64 bytes from node2 (10.101.1.12): icmp_seq=2 ttl=64 time=1.09 ms
64 bytes from node2 (10.101.1.12): icmp_seq=3 ttl=64 time=0.599 ms
64 bytes from node2 (10.101.1.12): icmp_seq=4 ttl=64 time=1.01 ms
^C
--- node2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3055ms
rtt min/avg/max/mdev = 0.599/0.826/1.090/0.227 ms
```

```
[leo@node2 ~]$ ping node1
PING node1 (10.101.1.11) 56(84) bytes of data.
64 bytes from node1 (10.101.1.11): icmp_seq=1 ttl=64 time=0.665 ms
64 bytes from node1 (10.101.1.11): icmp_seq=2 ttl=64 time=1.25 ms
64 bytes from node1 (10.101.1.11): icmp_seq=3 ttl=64 time=1.08 ms
^C
--- node1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2018ms
rtt min/avg/max/mdev = 0.665/0.998/1.247/0.247 ms
```
**les machines doivent avoir un nom**

```
[leo@node1 ~]$ hostname
node1.tp1.b2
```

```
[leo@node2 ~]$ hostname
node2.tp1.b2
```

**utiliser 1.1.1.1 comme serveur DNS**

```
[leo@node2 ~]$ sudo cat /etc/resolv.conf
search auvence.co tp1.b2
nameserver 10.33.10.2
nameserver 10.33.10.148
nameserver 10.33.10.155
nameserver 1.1.1.1
```

```
[leo@node1 ~]$ sudo cat /etc/resolv.conf
search auvence.co tp1.b2
nameserver 10.33.10.2
nameserver 10.33.10.148
nameserver 10.33.10.155
nameserver 1.1.1.1
```

**les machines doivent pouvoir se joindre par leurs noms respectifs**

```
[leo@node1 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.101.1.12     node2
```

[leo@node2 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.101.1.12     node1
```

[leo@node1 ~]$ ping node2
PING node2 (10.101.1.12) 56(84) bytes of data.
64 bytes from node2 (10.101.1.12): icmp_seq=1 ttl=64 time=0.603 ms
64 bytes from node2 (10.101.1.12): icmp_seq=2 ttl=64 time=1.09 ms
64 bytes from node2 (10.101.1.12): icmp_seq=3 ttl=64 time=0.599 ms
64 bytes from node2 (10.101.1.12): icmp_seq=4 ttl=64 time=1.01 ms
^C
--- node2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3055ms
rtt min/avg/max/mdev = 0.599/0.826/1.090/0.227 ms
```

```
[leo@node2 ~]$ ping node1
PING node1 (10.101.1.11) 56(84) bytes of data.
64 bytes from node1 (10.101.1.11): icmp_seq=1 ttl=64 time=0.665 ms
64 bytes from node1 (10.101.1.11): icmp_seq=2 ttl=64 time=1.25 ms
64 bytes from node1 (10.101.1.11): icmp_seq=3 ttl=64 time=1.08 ms
^C
--- node1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2018ms
rtt min/avg/max/mdev = 0.665/0.998/1.247/0.247 ms
```

**firewall**

```
sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

# I. Utilisateurs

## 1. Création et configuration

**Ajout d'un utilisateur**

```
[leo@node1 ~]$ sudo useradd toto -m -s /bin/sh -u 2000
```
```
[leo@node2 ~]$ sudo useradd toto -m -s /bin/sh -u 2000
```

**Ajout du groupe admins**

```
[leo@node1 ~]$ sudo groupadd admins
```
```
[leo@node2 ~]$ sudo groupadd admins
```

**visudo**

```
[leo@node1 ~]$ sudo visudo
[...]
%admins ALL=(ALL)        ALL
[...]
```
```
[leo@node2 ~]$ sudo visudo
[...]
%admins ALL=(ALL)        ALL
[...]
```

**Ajouter de l'utilisateur au groupe admins.**

```
[leo@node1 ~]$ sudo usermod -aG admins toto
```
```
[leo@node2 ~]$ sudo usermod -aG admins toto
```

## 2. SSH

**Génération de clés SSH**
```
$ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Your identification has been saved in C:\Users\lbour/.ssh/id_rsa.
Your public key has been saved in C:\Users\lbour/.ssh/id_rsa.pub.
```

**Dépôt de la clé ssh**
```
[toto@node1 ~]$ mkdir .ssh
[toto@node1 ~]$ cd .ssh
[toto@node1 .ssh]$ vim authorized_keys
```
```
[toto@node1 .ssh]$ chmod 600 /home/toto/.ssh/authorized_keys
[toto@node1 .ssh]$ chmod 700 /home/toto/.ssh/
```

**Maintenant plus besoin du mot de passe pour se connecter en SSH.**

# II. Partitionnement

```
$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0    8G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0    7G  0 part
  ├─rl-root 253:0    0  6.2G  0 lvm  /
  └─rl-swap 253:1    0  820M  0 lvm  [SWAP]
sdb           8:16   0    3G  0 disk
sdc           8:32   0    3G  0 disk
sr0          11:0    1 1024M  0 rom
```
```
$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```
```
$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
```
$ sudo pvs
  PV         VG Fmt  Attr PSize  PFree
  /dev/sda2  rl lvm2 a--  <7.00g    0
  /dev/sdb      lvm2 ---   3.00g 3.00g
  /dev/sdc      lvm2 ---   3.00g 3.00g
```

**Ajout des deux disques en un seul volume group.**

```
$ sudo vgcreate vgroup /dev/sdb
  Volume group "vgroup" successfully created
```
```
$ sudo vgextend vgroup /dev/sdc
  Volume group "vgroup" successfully extended
```
```
$ sudo vgs
 [...]
 vgroup   2   0   0 wz--n-  5.99g 5.99g
```

**Création de 3 logical volumes de 1 Go chacun.**

```
$sudo lvcreate -L 1G vgroup -n data
```
```
$ sudo lvs
  [...]
  data  vgroup -wi-a-----   1.00g
  data2 vgroup -wi-a-----   1.00g
  data3 vgroup -wi-a-----   1.00g
```
```
$ mkfs -t ext4 /dev/data/ma_data_frer
```

**Formatage des partitions en ext4**

```
$ sudo mkfs -t ext4 /dev/vgroup/data
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 0efad9af-73f6-4d34-a207-d2505fe642cc
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
```
$ sudo mkfs -t ext4 /dev/vgroup/data2
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: a73236fe-ef06-4feb-b5cb-9256257a3685
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```
```
$ sudo mkfs -t ext4 /dev/vgroup/data3
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 814ed73f-5ea6-4a03-93d1-907bc3e88e80
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

**Montage des partitions**

```
$ sudo mkdir /mnt/part1
```
```
$ sudo mkdir /mnt/part2
```
```
$ sudo mkdir /mnt/part3
```
```
$ sudo mount /dev/vgroup/data /mnt/part1
```
```
$ sudo mount /dev/vgroup/data2 /mnt/part2
```
```
$ sudo mount /dev/vgroup/data3 /mnt/part3
```

**Montage automatique au boot de la machine**

```
$ sudo nano /etc/fstab
```
```
/dev/data/vgroup        /mnt/data               ext4    defaults        0 0
/dev/data/vgroup        /mnt/data2              ext4    defaults        0 0
/dev/data/vgroup        /mnt/data3              ext4    defaults        0 0
```

# III. Gestion de services

## 1. Interaction avec un service existant

```
$ systemctl is-active firewalld.service
active
```
```
$ systemctl is-enabled firewalld.service
enabled
```

## 2. Création de service

### A. Unité simpliste

```
$ sudo touch /etc/systemd/system/web.service
```
```
$ sudo nano /etc/systemd/system/web.service
```

**Contenu de web.service**
```
[Unit]
Description=Very simple web service

[Service]
ExecStart=/bin/python3 -m http.server 8888

[Install]
WantedBy=multi-user.target

```

**Ouverture du port 8888 et lancement de web-service**
```
$ sudo firewall-cmd --add-port=8888/tcp --permanent
success
```
```
$ sudo systemctl daemon-reload
```
```
$ sudo systemctl start web
```
```
$ sudo systemctl enable web
Created symlink /etc/systemd/system/multi-user.target.wants/web.service → /etc/systemd/system/web.service.
```
```
$ sudo systemctl status web
● web.service - Very simple web service
   Loaded: loaded (/etc/systemd/system/web.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2021-09-25 22:41:04 CEST; 8s ago
 Main PID: 23732 (python3)
    Tasks: 1 (limit: 11385)
   Memory: 10.3M
   CGroup: /system.slice/web.service
           └─23732 /bin/python3 -m http.server 8888

Sep 25 22:41:04 node1.tp1.b2 systemd[1]: Started Very simple web service.
```

**Vérification**

```
$ curl 10.101.1.11:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="bin/">bin@</a></li>
<li><a href="boot/">boot/</a></li>
<li><a href="dev/">dev/</a></li>
<li><a href="etc/">etc/</a></li>
<li><a href="home/">home/</a></li>
<li><a href="lib/">lib@</a></li>
<li><a href="lib64/">lib64@</a></li>
<li><a href="media/">media/</a></li>
<li><a href="mnt/">mnt/</a></li>
<li><a href="opt/">opt/</a></li>
<li><a href="proc/">proc/</a></li>
<li><a href="root/">root/</a></li>
<li><a href="run/">run/</a></li>
<li><a href="sbin/">sbin@</a></li>
<li><a href="srv/">srv/</a></li>
<li><a href="sys/">sys/</a></li>
<li><a href="tmp/">tmp/</a></li>
<li><a href="usr/">usr/</a></li>
<li><a href="var/">var/</a></li>
</ul>
<hr>
</body>
</html>
```

### B. Modification de l'unité

**Création de l'utilisateur web**

```
$ sudo adduser web
```

**Modification de web.service**

```
[Unit]
Description=Very simple web service

[Service]
ExecStart=/bin/python3 -m http.server 8888
User=web
WorkingDirectory=/srv/serveur

[Install]
WantedBy=multi-user.target
```

**Vérification**

```
$ curl 10.101.1.11:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="monfichier">monfichier</a></li>
</ul>
<hr>
</body>
</html>
```
