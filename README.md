# centOS
setting up centOS with networking and administrating


## Content


[Requirements](#requirements)

[Install centOS](#install-centos)

[putty](#putty)

[ssh in centos](#ssh-in-centos)

[user permissions](#user-permissions)

[close access for root(log in)](#close-access-for-rootlog-in)

[install repo epel and remi](#install-repo-epel-and-remi)

[network management](#network-management)

[autorun servers  systemctl](#autorun-servers-systemctl)

[top htop atop iotop iostat](#top-htop-atop-iotop-iostat)

[fstab](#fstab)

[iptables](#iptables)

[install local cashe dns server](#install-local-cashe-dns-server)


## Requirements

install Virtual box

download centos 8.  32-64. boot.iso

In virtualBox create Vm 1024Ram 8Gb Hdd

settings->network->adapter1->network bridge

Then run machine ->chose iso file which was downloaded previosly

[Content](#content)

## Install centOS

run Vm

configure and install(enable network)

`dhclient -v ` ----get network settings(create)

`route -n` ---show network

`ip a `   -------show netw devices

`nmtui`  ---run programm to redact network -> edit conn->ip4->set to manual-> paste IP(adresses) from `ip a`

gateway(192.168.1.1) DNS (8.8.8.8) and enable automatically connect  then save and exit 

and restart`systenctl restart NetworkManager`

dnf allow install multiple software

`mc` - midnight commander for faast navigation on fylesystem

wget` - utility for fast downloading files

`net-tools `- utils for network

`dnf install mc wget git net-tools -y`

[Content](#content)

## putty

https://www.fosshub.com/KiTTY.html ---download without compression https://ttyplus.com/downloads/ download installation package and install

https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html   ->download puttygen.exe,  pageant.exe

run MTPuTTY chose path to kitty

server-> server name =cld.livelinux.net->protocol=ssh->port=22->username and passsword

run puttygen.exe generate key ->set key comment=your mail-> then save public and private key

run pagent.exe -> add key(private)

[Content](#content)

## ssh in centos

`systemctl status sshd -l`

`systemctl stop firewalld`

`systemctl disable firewalld`



MTPuTTY
->server ->server name =your IP->protocol=ssh->port=22->username and password then 2click on session

open puttygen and load private key and copy `key`

in console create `touch authorized_keys` then `nano autorized_keys` then right button mouse out of nano to exit ctrl+x(Y to save then enter)

run `ssh-keygen`

check: `nano /root/.ssh/authorized_keys

`nano /etc/ssh/sshd_config` navigate using ctrl+w  ->uncomment port and =33555->permit root login =yes -> password authentification=no  ->pubkeyAuthentification =yes ->save ctrl+o

then `systemctl restart sshd`

`netstat -tulnp | grep ssh`

then create new server with port 33555

[Content](#content)

## user permissions

create group `groupadd dev` `groupadd development` 

check group `cat /etc/group |grep dev`

`useradd -m dev -G development -s /bin/bash -p pass -d /home/development -g dev`

create user (-m)-create home directory,  (dev )-username, (-G)-add to group, (development)-group name, (-s)-start console, (/bin/bash)-bash, (-p)-password, (pass)-password for dev, 
(-d) home directory, (/home/development)-directory, (-g)-add to group

change password `passwd username`

change group `usermod -aG wheel dev`   wheel -group superusers centos os(add user dev to group wheel)

to check user groups `groups username(dev)` or `groups dev`

`su dev`  ** switch to sudo mode with user dev

then look inside secure folder -> sudo nano /root/.ssh/authorized_keys  -> and tap password for dev

work as root `sudo -s`



**********
Why you shouldn`t use root user 
because of brutforce attacks have as a goal root user
we give some permissions to another users and close opportunity to log in as root
********

[Content](#content)

## close access for root(log in)

`nano /etc/ssh/sshd_config` PermitRootLogin  no ->save

`su dev `***change user to dev

`mkdir /home/development/.ssh/`

`sudo cat /root/.ssh/authorized_keys`

copy key `nano /home/development/.ssh/authorized_keys` paste key -> save

check log `sudo tail -f /var/log/audit/audit.log` or `journalctl -u sshd`

`chmod g-w /home/development/`

`chmod 700 ~/.ssh`

`chmod 600 ~/.ssh/authorized_keys`

to check user permissions `ls -l /home/development/.ssh`

try to connect with dev change user in putty to restart with change root login `systemctl restart sshd`

[Content](#content)

## install repo epel and remi

`dnf repolist `**show installed repositories

`sudo dnf install epel-release -y  `**install repo epel

`cat /etx/yum.repos.d  `**files repo

`ls -alh /etx/yum.repos.d `** show directory with files repo

epel work in parallel

remi rewrite repos

sudo dnf install -y  http://rpms.remirepo.net/enterprise/remi-release-8.rpm**install repo remi

dnf update **update system

[Content](#content)

## network management

`sudo systemctl start NetworkManager`

`systemctl status NetworkManager`

`nmcli general status `** show interfaces to connect

`nmcli radio wifi off `** turn off wifi

`nmcli radio wifi  `**check status

`nmcli connection show `**show connection

`nmcli device show `** show devices network

sudo nmcli connection add con-name "dhcp" type ethernet ifname enp0s3 ** save connection

`sudo nmcli connection del dhcp `** delete connection 

`ip a `**show ip 'inet'

sudo nmcli connection add con-name "static" ifname enp0s3 autoconnect no type ethernet ip4 192.... gw4 192.168...** save connection

`nmcli con up "static"  `** activate interface

sudo nmcli connection modify "static" ipv4.dns 8.8.8.8

`date `**show time

`sudo dnf install chrony `**

`nano /etc/chrony.conf `**show config  and if need change

sudo timedatectl set-timezone Europe/Moscow

sudo systemctl start chronyd

sudo systemctl enable chronyd

`chronyc sources `** check active server

[Content](#content)

## autorun servers  systemctl

systemctl status firewalld

systemctl start firewalld

systemctl enable firewalld** enable autorun firewall

systemctl disable firewalld** disable autorun firewall

systemctl is-enabled firewalld**check

systemctl stop firewalld

systemctl restart firewalld

systemctl list-unit-files | grep enabled

systemctl list-unit | awk '{print $1;system("systemctl is-enabled" $1)}'


systemd - demon to run other demon using systemctl

unit - describe service in text where defined operation before run and after run

/etc/systemd/system - admin units

/usr/lib/system/system - units installed from rpm package

/run/systemd/system - highest priority units(runtime)

[Content](#content)

## top htop atop iotop iostat

top - show processes
sudo install htop
htop
shift + h ***show threads
Load average 0.0 0.0 0.0(1st =middle for 1 min, 2d = middle for 5 min, 3d = middle for 15min) by CPU sum processes

iotop - disk
 sync; dd if=/dev/zero of=tempfile bs=2M count=2048; sync  ***create file 3 gb and check write speed


lsof  - list of open files we can find scripts

lsof | less
man lsof
lsof /usr/sbin/mysqld

inode - data about files

if we don`t have inode we can`t work with files that means we don`t have space
check filesystem `df -hT`
check inodes `df -hTi`

delete
rm -rf /mnt/junkdirectory/ *** garantee delete files without consuming a loy of CPU (r means recursively)
rm /mnt/junkdirectory/*   ***delete but consume CPU

logrotate
nano /etc/logrotate.d/nginx

```sh

/var/log/nginx/*.log.*.gz {
daily
missingok
rotate 14
compress
delaycompress
notifempty
create 0640 nginx nginx
sharedscripts
prerotate
                 if [-d /etc/logrotate.d/httpd-prerotate ]; then \
                 run-parts /etc/logrotate.d/httpd-prerotate; \
                 fi \
endscript
postrotate
                   invoke-rc.d nginx rotate >/dev/null 2>&1
endscript
```

[Content](#content)


## fstab

fstab - mont directory and remote

dhclient
ip a (inet)
route -n (gateway)

dnf config-manager --set-enabled powertools
 dnf repolist
dnf install fuse-sshfs -y

#server1

mkdir /mnt/backup

#server2

mkdir /mnt/backup 
ssh-keygen
cat /root/.ssh/id_rsa.pub  -> copy

#server1

nano /home/development/.ssh/authorized_keys -> paste key
sshfs dev@ip_server#1:/mnt/backup/ /mnt/backup/ -p 33555
ssh dev@ip_server#1-p 33555
sudo touch /mnt/backup/TEST

#server2

ls -la /mnt/backup 
 after restart server it not mount automatically
we use fstab `fstab mount -all`

nano /etc/fstab
-> put
sshfs#dev@ip_server#1:/mnt/backup/ /mnt/backup/ fuse delay_connect,port=55567, defaults, allow_other,noempty 0 0
->save

nano /etc/rc.local
->put
mount -a
->save

[Content](#content)

## iptables

iptables -t<table><command><chain>[number]<condition><action>

DROP =ignore, ACCEPT,REJECT

we have only 4 tables

-A = add

-P =by default

iptables -t filter -A INPUT 10 -s 10.10.10.0/255.255.255.0 -j DROP

iptables -L

iptables -t filter -A INPUT -p tcp --dport 33555 -j ACCEPT*** allow connect to ssh

iptables -t filter -P INPUT DROP*** by default block all income traffic

iptables -t filter -P INPUT ACCEPT**allow income traffic

------------------------
nftables
------------------------

nft list ruleset

we don`t have tables by default -> we can create IP IP6 INET ARP BRIDGE NETDEV

nft add table inet my

nft add chain inet my filter_chain { type filter hook input priority 0 \; }

nft add rule inet my filter_chain tcp dport ssh accept   ***add to end of chain 

nft insert rule inet my filter_chain tcp dport http accept ***add to begin of chain 

nft insert rule inet my filter_chain index 1 tcp dport nfs accept ***add to index of chain 

--------------
fdisk  array disk
--------------

add to VM 2 virtual disk 

fdisk ***show disks

/dev/md0 = name array
 --level=1    **type
--raid-devices=2 **quantity disks

mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdc /dev/sdb

lsblk
mkfs.ext4 /dev/md0  **create ext4 system

mdadm --detail /dev/md0
mount /dev/md0 /mnt **mount
df -h
nano fstab
-> `to mount after restart` put->
/dev/md0 /mnt           ext4        defaults 0 1
->save

------------------------------
reset root password
------------------------------

restart server
in grub `y`
then in menu find where root -> ro change to `rw\ init=/sysroot/bin/sh ...what where before` and ctrl+x
then after restart

`chroot /sysroot`
passwd root
->enter new passwd 2 times
touch /.autorelabel
->restart

/sbin/reboot -f

[Content](#content)

## install local cashe dns server


systemd-resolved 127.0.0.53 53

resolvectl status

nano /etc/systemd/resolved.conf 
->put
DNS=8.8.8.8.8.8.4.4
->
uncomment
DNSEC=allow-downgrade
DNSOverTLS=opportunistic
->save

systemctl restart systemd-resolved.service

nano /etc/resolv.conf
->set nameserver first
127.0.0.53 ->save
 
nscd - service cashe

dnf install -y nscd

systemctl start nscd

systemctl enable nscd

 nscd -g
to check time with cashe and without use `time wget -q -O/dev/null google.com`

[Content](#content)
