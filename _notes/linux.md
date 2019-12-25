---
layout: post
---

* TOC
{:toc}

# /etc/network/interfaces

```sh
allow-hotplug eth0
iface eth0 inet static
      address 10.1.1.125
      netmask 255.0.0.0
      gateway 10.1.1.1
```

# wireguard

Use: https://github.com/mina-alber/wireguard-ansible
https://blog.morad-edwar.com/your-own-wireguard-vpn-server-in-5-minutes/

ansible-playbook -i hosts_inventory wireguard.yml -u docker --become-method=sudo --become-user=root -b




# bind
Authoritive: answer to resource records that are part of their zones only. category includes both primary (master) and secondary (slave) nameservers
Recursive: offer resolution services, not authoritive for any zone. answers all resolutions cached in memory for a fixed period of time, which is specified by the retrieved resource record

BIND - contains named, rndc, dig
/etc/named.conf - main config file
/etc/named - auxiliary directory for config files that are included in the main config file

Installing BIND in a chroot environment:
 yum install bind-chroot
 systemctl status named
 systemctl stop named
 systemctl disable named
 systemctl enable named-chroot
 systemctl start named-chroot
 systemctl status named-chroot

Following are commonly used / found in /etc/named.conf
 acl
   acl black-hats {
     10.0.2.0/24;
     192.168.0.0/24;
     1234:5678::9abc/24;
   };
   acl red-hats {
     10.0.1.0/24;
   };
   options {
     blackhole { black-hats; };
     allow-query { red-hats; };
     allow-query-cache { red-hats; };
   };
 include "file-name"
 zone
 controls
 key
 logging
 server
 trusted-keys
 view
 command //,#,/*,*/

Named service zone files:
 /var/named/
 /var/named/slaves/
 /var/named/dynamic/
 /var/named/data/

Common directives:
 $INCLUDE
 $ORIGIN
 $TTL

Common resource records:
 A - address record specifies an IP address to be assigned to a name. 
 CNAME - canonical name maps one name to another. 
   server1 IN A 10.0.1.5
   www     IN CNAME server1
 MX - mail exchange record specifies where the mail sent to a particular namespace controlled by this zone should go 
 NS - nameserver record announces authoritative nameservers for a particular zone. 
   IN NS nameserver-name
 PTR - reverse name resolution
 SOA - start of authority. The first resource record in a zone file. 
   @ IN SOA primary-name-server hostmaster-email (
   serial-number
   time-to-refresh
   time-to-retry
   time-to-expire
   minimum-TTL )

Example zone file:
 zone "1.0.10.in-addr.arpa" IN {
   type master;
   file "example.com.rr.zone";
   allow-update { none; };
 }

# blkid
use blkid to find identifier for device
ex. blkid /dev/sda1

# btrfs
mkfs.btrfs /dev/device # to make
mkfs.btrfs /dev/device1 /dev/device2 /dev/device3 /dev/device4 # raid 10
mkfs.btrfs -m raid0 /dev/device1 /dev/device2 # stripe without mirroring

mount /dev/device /mount-point

btrfs filesystem resize +2g,max,-2g,20g /mount-point # resize
btrfs device scan # scan all devices
btrfs device add /dev/device2 /mount-point # adding new device to an btrf fs

convert existing non-raid file system to a raid
 mount /dev/sdb1 /mnt
 btrfs device add /dev/sdc1 /mnt
 btrfs balance start -dconvert=raid1 -mconvert=raid1 /mnt

btrfs device delete /dev/sdc /mnt # remove

remove from raid-1 failed device
 mount -o degraded /dev/sdb /mnt
 btrf device delete missing /mnt

registering in /etc/fstab
 /dev/sdb    /mnt    btrfs
 device=/dev/sdb,device=/dev/sdc,device=/device/sdd,device=/dev/sde    0

# cdrom mount
mount -o ro,loop Fedora-14-x86_64-Live-Desktop.iso /media/cdrom
umount /media/cdrom


# compression tools
gzip
bzip2
bunzip2
tar

# disk quotas
alerts an admin before a user conusmes too much disk space or a partition becomes full
enabling quotas:
 vim /etc/fstab
   /dev/sda2 /home ext3 defaults,usrquota,grpquota 1 2
 quotacheck -cug /home
managing disk quotas
 quotaoff -vaug
 quotaon -vaug
 quotaon -vug /home

# dnsmasq
A DNS cacher and DHCP server

DNS queries will resolve first with dnsmasq, only checking external servers if dnsmasq cannot resovle the query.

# dovcot / postfix

Using dovecot, a pop3 / imap server:

vim /etc/dovecot/dovecot.conf
 protocols = imap imaps pop3 pop3s

systemctl restart dovecot
' enable dovecot

To enable ssl:
 bash /usr/libexec/dovecot/mkcert.sh

postfix is default
sendmail is considered deprecated

postfix stores config files in /etc/postfix
 access - used for access control
 main.cf - main config file
 master.cf - how postfix interacts with other processes to accomplish mail delivery
 transport - maps email address to relay hosts

# /etc/sysconfig/network-scripts
All network config files are stored here.
eno1 - onboard / bios supplied index numbers
ens3 - PCI express hotplug src nubers
ensp3s0 - adaptors in pci slot with index number on adapter

# ext3
mkfs -t ext3 -U 7cd65de3-e0be-41d9-b66d-96d749c02da7 /dev/sda8
tune2fs -U 7cd65de3-e0be-41d9-b66d-96d749c02da7 /dev/sda8
tune2fs -j block_device # to convert ext2 fs to ext3

# ext4
mkfs.ext4 /dev/device
mkfs.ext4 -E stride=16,stripe-width=64 /dev/device
mount /dev/device /mount/point # to mount
mount -o acl,user_xattr /dev/device /mount/point
resize2fs /mount/device size # resizing

# file-system check tools
fsck tool
e2fsck #ext2,ext3,ext4
xfs_repair
btrfsck


# fs-cache
  Persistet local cache that can be used by file systems to take data retrieved from over the network and cache it on local disk. 
  tune2fs -o user_xattr /dev/device
  mount /dev/device /path/to/cache -o user_xattr
  service cachefilesd start
  chkconfig cachefilesd on
  mount nfs-share:/ /mount/point -o fsc # using fs-cache with NFS


# httpd

Installing:
 yum install httpd
 systemctl start httpd.service
 systemctl enable httpd.service

Config files:
 /etc/httpd/conf/httpd.c
 /etc/httpd/conf.d

Loading a module:
 edit the config file in /etc/httpd/conf.d adding: LoadModule ssl_module modules/mod_ssl.so

Enabling ssl:
 yum install mod_ssl openssl
 mv key_file.key /etc/pki/tls/private/hostname_key
 mv certificate.crt /etc/pki/tls/certs/hostname.crt
 vim /etc/httpd/conf.d/ssl.conf
   SSLCertificateFile /etc/pki/tls/certs/hostname.crt
   SSLCertificateKeyFile /etc/pki/tls/private/hostname.key

Generating a new key / cert:
 yum install crypto-tools
 openssl req -x509 - new - set_serial number - key hostname.key - out hostname.crt
 or 
 genkey

# hwclock

Setting the HW clock:

hwclock --set --date "dd mmm yyy HH:MM"
hwclock --systohc
hwclock --systohc --localtime

# intro to linux containers
Mount namespaces - isolate set of file sys mount points
UTS - isolate nodename and domainname returned by uname() sys call
IPC - isolate IPC resources such as System V IPC objects and POXIS message queues
PID - allow processes in different containters to have the same PID
Network - provide isolation of network controllers
cgroups - to group processes for the purpose of system resource management
selinux - provides secure separation of containers by applying selinux policy and labels
management interfaces - RHEL provides the Docker aplication as a main management tool for linux containers

# ip routing
Add static routes:
ip route add 192.168.3.0/24 via 192.168.0.1 dev eth1
ip route add 192.168.4.0/24 via 192.168.2.1 dev team0

Permanent:
vi route-eth1
   ADDRESS0=192.168.3.0
   NETMASK0=255.255.255.0
   GATEWAY0=192.168.0.1
vi route-team0
   ADDRESS0=192.168.4.0
   NETMASK0=255.255.255.0
   GATEWAY0=192.168.2.1

# kdump

Install:
yum install kexec-tools system-config-kdump

Edit grub to:
 GRUB_CMDLINE_LINUX="rd.md=0 rd.dm=0 rd.lvm.lv=rhel/swap $([ -x /usr/sbin/rhcrashkernel-param ] && /usr/sbin/rhcrashkernel-param || :) rd.luks=0 vconsole.keymap=us rd.lvm.lv=rhel/root rhgb quiet"
 grub2-mkconfig -o /boot/grub2/grub.cfg
 grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg # on EUFI-based systems

Change params:
vim /etc/kdump.conf # to change params

Start:
systemctl enable kdump.service
systemctl start kdump.service

Analyzing crashes:
yum install crash
crash /var/crash/<timestamp>/vmcore /usr/lib/debug/lib/modules/<kernel>/vmlinux

# kerberos
TLDR: get authentication / security with a ticket-system for non-secure programs / protocols

Two software packages:
krb5-server # the actual server
krb5-workstation # client

Several tools are also installed:
kinit # obtains and caches TGT
kdestroy # destroys tickers stored in credential cache
klist # list cached tickers
kpasswd # changes principals pass
kadmin # administers kerberos database via kadmind daemon
kadmin.local # same as kadmin but performs operations directly on the KDC database

Configure a client to authenticate using Kerberos:
yum -y install krb5-workstation
vi /etc/krb5.conf # ensure file has directives set, first three disable DNS lookups and set default kereros realm, next set defines hostnames for the KDC and admin servers, last defines mapping bewtween DNS domains and kerberos realms
   dns_lookup_realm = false
   dns_lookup_kdc = false
   default_realm = EXAMPLE.COM
   [realms]
      EXAMPLE.COM = {
         kdc = server2.example.com
         admin_server = server2.example.com
      }
   [domain_realm]
      example.com = EXAMPLE.COM
      .example.com = EXAMPLE.com

kadmin -p root/admin # login as th eorot principal and add server1 as a host principal to the KDC database
   addprinc -randkey host/server1.example.com
   ktadd -k /etc/krb5.keytab host/server1.example.com # whle logged in, extract principals key and store it locally in a keytab file called krb5.keytab in the /etc dir

authconfig --enablekrb5 --update
vi /etc/ssh/ssh_config # make sure client config file have the two lines set
   GSSAPIAuthentication yes
   GSSAPIDelegateCredentials yes

kinit # login as user1 and execute kinit command to obtain TGT from the KDC
klist # list the TGT details

# linux root file structure
/boot 
/dev # device nodes that represent device types
/etc # config files local to the machine, no binaries
/media # mount points for usb, storage media, etc
/mnt # dir reserved to temporarily mounted points
/opt # sfotware and add-on packages not part of standard default installation
/proc # files to extract info from kernel or setn info to it
/srv # site-specific data served by a RHEL system
/sys # utilizes new sysfs virtual file system specific to the kernel
/usr # files shared across multiple machines
 /bin # binaries
 /etc
 /games
 /include # C header files
 /kerberoes
 /lib # object files and libraries not designed to be usitilied by shell scripts or users
 /libexec
 /sbin # sys admin binaries sych as rebooting, restoring, recovering, repairing, etc.
 /share # not arch specific files
 /src
 /tmp # same as /var/tmp
/var 

# listing pci, usb and cpu info
lspci -m
lsusb
lscpu

# mdadm / raid
0 - striping
1 - mirroring 
4 - parity
5 - most common. parity over min of 3 hdd's
6 - 
10 - 
Linear RAID
mdadm - command line tool
dmraid - used to manage device-mapper RAID sets

# mtp
android support for thunar / nautilus:
sudo apt-get install gvfs-backends

# mutt
To tag everything:
shift+T
~A
;(then your command, y for archive?)

# net analysis tools
ss
ip
dropwatch
ethtool
/proc/net/snmp


# nmcli
Configure and active network using NetworkManager CLI:
nmcli dev status
nmcli con add type Ethernet ifname eth1 con-name eth1 ip4 192.168.0.121/24 gw4 192.168.0.1

# ntp
Installing:
config file /etc/ntp.conf
yum install ntp
systemctl enable ntpd

Setting up a localtime:
ln -sf /usr/share/zoneinfo/America/Toronto /etc/localtime

Commands in ntp:
 discard
 peer address
 server address
 broadcast address
 manycastclient address
 broadcastclient
 manycastserver address
 multicastclient address
 burst
 iburst
 key number
 ex:
   server 192.168.1.1 key 10
   broadcast 192.168.1.255 key 20
   manycastclient 239.255.254.254 key 30
 minpoll value and maxpoll value
 prefer
 ttl value
 version value
 SYNC_HWCLOCK=yes
   hwclock --systohc (to update hw clock from sys clock)

Using a predefined NTP polling client:
   yum install -y ntp
   cat /etc/ntp.conf
      server 0.rhel.pool.ntp.org iburst
   systemctl enable ntpd
   systemctl start ntpd

Configure ntp server and polling client:
   yum install -y ntp
   vi /etc/ntp.conf # comment out all servers and add
      server 127.127.1.0
   systemctl enable ntpd
   firewall-cmd —permanent —ad-service ntp
   firewall=cmd —reload
   systemctl start ntpd
   ntpq -p # to check that it’s online

Configuring an NTP peer:
   vi /etc/ntp.conf # comment out all servers and add
      peer server1.example.com
   systemctl enable ntpd
   firewall-cmd —permanent —add-service ntp
   firewall-cmd —reload
   systemctl restart ntpd

Configure a broadcast server and client:
Server2:
   vi /etc/ntp.conf
      server 0.rhel.pool.ntp.org iburst
      server 1.rhel.pool.ntp.org iburst
      server 2.rhel.pool.ntp.org iburst
      server 3.rhel.pool.ntp.org iburst
      broadcast 192.168.0.255
   systemctl enable ntpd
   firewall-cmd —permanent —add-service ntp
   firewall-cmd —reload
   systemctl restart ntpd
Server1:
   vi /etc/ntp.conf # comment out servers
      broadcastlient
      disable auth
  systemctl restart ntpd

# oprofile
A low overhead, system-wide performance monitoring tool. Uses performance monitoring hardware on the processor to retrieve info about kernel and execs on the system. 

Commands:
 ophelp
 opimport - convert sample db files from foreign binary format to native format for system
 opannotate - creates annotate source for exec if app compiled with debugging symbols
 opcontrol - configs what data is collected
 operf - tool to be used in place of opcontrol for profiling
 opreport - retrieves profile data
 oprofiled - runes daemon to periodically write sample data to disk

operf vs opcontrol: operf is the prefered way, uses linux performance events subsystem and does not require the oprofile kernel driver.

Using operf: 
 operf options range command args --system-wide --pid=PID --vmlinux=vmlinux_path
 operf --events=event1,event2,event3...
 operf --events=event-name:sample-rate:unit-mask:kernel:user
samples can be put into separate threads or cpu using: --separate-thread, --separate-cpu

To setup in legacy mode:
 opcontrol --setup --vmlinux=/usr/bin/debug/lib/modules/’uname -r’/vmlinux

To not monitor the kernel:
 opcontrol --setup --no-vmlinux

Unit masks are listed in the ophelp command
to setup, ex.
 opcontrol --event=event-name:sample-rate:unit-mask:1:0

To save:
 opcontrol --save=name
Dump:
 opcontrol --dump

opreport provides an overview of all the executables being profiled

opannotate tries to match samples for particular instructions to the corresponding lines in the source code, ex:
 opannotate --search-dirs src-dir --source executable

when using in legacy mode, /dev/oprofile/ dir is used to store file system info for oprofile

oprofile can be used natively with JVM, ex (added to end of jvm instance):
 -agentlib:/usr/lib64/oprofile/libkvmti_oprofile.so

gui: oprofile-gui

# performance analysis tools
uptime
dmesg | tail
vmstat 1
mpdstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
tuna
‘/proc/’
ps
sar
tuned
tuned-adm
perf
turbostat
iostat
irqbalance
numastat
numad
systemtap
oprofile
valgrind

# power management tools
powertop
diskdevstat
netdevstat
tuned
tuned-adm

# /proc/sys/vm/
Physical memory is managed in chunks called pages. Location of each page is mapped to a virtual location so that the processor can access the memory. By default, a page is about 4 KB. Static huge pages cna be configured up to sizes of 1GB. Transparent huge pages are an alternative to static huge pages. They are 2 MB in size and are enabled by default.

valgrind --tool=memcheck application
valgrind --tool=cachegrind application

To overcommit memory:
echo 1 > /proc/sys/vm/overcommit_memory or sysctl vm.overcommit_memory=1

Virtual memory parameters:
dirty_ratio - percentage. when modified system beginds writing modifications to disk with the pdflush operation
dirty_background_ratio - % begins writing modifications to disk in the background
overcommit_memory
overcommit_ratio - percentage of ram considered when overcommit_memory is set to 2
max_map_count - max num of mem map areas process can use
min_free_kbytes - min number of kb to keep free across the system
oom_adj
swappiness - controls degree to which the system swaps

File sys parameters:
aio-max-nr - max allowed numbe rof events in all active async i/o contexts
file-max - max numbe rof file handles alocated by the kernel

Kernel params:
msgmax - max allowed size in bytes of any single message in a message queue
msgmnb - deifnes max size in bytes of a single message queue
msgmni - defines max numbe rof message queue identifiers
shmall - total amount of shared memory in pages that can be used on the sys at one time
shmmax - max size of single shared mem segment by kernel in pages
shmmni - defines sys-wide max number of shared memory segments
threads-max - defines system-wide max number of threads available to the kernel at one time

# repairing disks

Repairing damaged extended file system:
unmount /mnt/ext
e2fsck /dev/vg10/lvolext4
dumpe2fs /dev/vg10/lvolext4 | grep superblock # show list of superblock locations for this file system
fsck -b 32768 /dev/vga10/lvolext4
mount /mntext4

Repairing xfs file system:
unmount /mntxfs
xfs_repair /dev/vg10/lvolxfs
mount /mnt/xfs

# rhel-network bonding
Configure interface bonding by editing files:
uuidgen eth2
uuidgen eth3
cd /etc/sysconfig/network-scripts
vi ifcfg-bond0
   DEVICE=bond0
   NAME=Bond
   BONDING_MASTER=yes
   BONDING_OPTS=”mode=balance-rr”
   ONBOOT=yes
   BOOTPROTO=none
   IPADDR=192.168.1.110
   NETMASK=255.255.255.0
   GATEWAY=192.168.1.1
   IPV4_FAILURE_FATAL=no
   IPV6INIT=no
vi ifcfg-eth2
   DEVICE=eth2
   NAME=eth2
   UUID=uuidhere
   TYPE=Ethernet
   ONBOOT=yes
   MASTER=bond0
   SLAVE=yes
vi ifcfg-eth3
   DEVICE=eth3
   NAME=eth3
   UUID=uuidhere
   TYPE=Ethernet
   ONBOOT=yes
   MASTER=bond0
   SLAVE=yes

Configure via nmcli:
   modprobe bonding
   nmcli con add type bond con-name bon0 ifname bon0 mode balance-rr ipv5 192.168.1.120/24 gw4 192.168.1.1
   nmcli con add type bond-slave ifname eth2 master bond0  
   nmcli con add type bond-slave ifname eth3 master bond0  
   nmcli con up bond0

# rhel-network ipv6
Configure via files:
[server1]:
   IPV6INIT=yes
   IPV6ADDR=2602:306:cc2d:f592::A/24
   IPV6_DEFAULTGW=2602:306:cc2d:f591::1
[server2]:
   IPV6INIT=yes
   IPV6ADDR=2602:306:cc2d:f592::B/24
   IPV6_DEFAULTGW=2602:306:cc2d:f591::1

# rhel-network teaming
Via files:
yum -y install teamd
modprobe team
uuidgen eth4
uuidgen eth5
vi ifcfg-team0
   DEVICE=team0
   NAME=team0
   DEVICETYPE=Team
   TEAM_CONFIG=’{”runner”: {”name”:”activebackup”}, “link_watch”:{”name”:”ethtool”}}’
   ONBOOT=yes
   PREFIX=24
   BOOTPROTO=none
   IPADDR0=192.168.2.110
   NETMASK0=255.255.255.0
   GATEWAY0=192.168.2.1
vi ifcfg-eth4
   DEVICE=eth4
   NAME=eth4
   UUID=uuihere
   DEVICETYPE=TeamPort
   ONBOOT=yes
   TEAM_MSATER=team0
   TEAM_PORT_CONFIG’{”prio”:9}’
vi ifcfg-eth5
   DEVICE=eth5
   NAME=eth5
   UUID=uuihere
   DEVICETYPE=TeamPort
   ONBOOT=yes
   TEAM_MSATER=team0
   TEAM_PORT_CONFIG’{”prio”:10}’

Via nmcli:
   modprobe team
   nmcli con add type team con-name team0 ifname team0 ip4 192.168.2.120/24 gw4 192.168.2.1
   nmcli con add type team-slave con-name eth4 ifname eth4 master team0
   nmcli con add type team-slave con-name eth5 ifname eth5 master team0

# rhel-network tools
Commands:
ifdown / ifup
ip # replaces deprecated ifconfig and netstat

NetworkManager tools:
nmcli
nmtui

# rndc
rdnc - utility is a command line tool that allows you to administer the named service, both locally and from a remote machine.

relevant files
 /etc/named.conf
 /etc/rndc.conf
 /etc/rndc.key
rndc status
rndc reload
rndc reload localhost
rndc reconfig
rndc freeze localhost
rndc thaw localhost

rndc sign localhost #update dnssec keys and sign the zone
 zone "localhost" IN {
   type master;
   file "named.localhost";
   allow-update { none; };
   auto-dnssec maintain;
 };
rndc validation on #enables the DNSSEC validation
rndc querylog #enables query logging

dig name NS #looking up a name server
dig name A
dig -x address


# snapper
snapper -c home_volume create-config -f "lvm(xfs)" /home # LVM
snapper -c home_volume create-config -f btrfs /home # BTRFS
snapper -c home_vol delete snapshot_number
snapper -c home_vol list
snapper -c home_vol status 1..5
snapper -c home_vol diff 1..5

# swap space
swapoff -v /dev/VolGroup00/LogVol01
lvresize /dev/VolGroup00/LogVol01 -L +2G
mkswap /dev/VolGroup00/LogVol01
swapon -v /dev/VolGroup00/LogVol01

other

dd if=/dev/zero of=/swapfile bs=1024 count=65536
mkswap /swapfile
swapon /swapfile
vim /etc/fstab
 swapfile swap swap defaults 0 0


# /sys/block/hda/queue/scheduler
io schedulers

deadline - provide guaranteed latency for requests from the point at which requests reach the i/o scheduler
cfq - divides processes intwo three separate classes: real time, best effort and idle
noop - FIFO algorithm

To set default io scheduler:
 edit /etc/grub2.conf and add: elevator=scheduler_name
 echo cfq > /sys/block/hda/queue/scheduler

# system monitoring tools
ps aux - display info about running processes
top - displays real-time list of processes that are running on the system
free - display amount of free and used memory on the system (free -m is show in megabytes)
lsblk - display llist of available block devices
blkid - display info block ids
findmnt - displays target points / mounted file systems
df - display detailed report on disk space usage
du - display amount of space used by files in directory (du -sh shows total in directory size)
lscpi - shows info about PCI buses and devices attached to them
lsusb - shows info on USB buses / devices
lspcmcia - list of all PCMCIA devices
lscpu - info about CPU


# tigervnc
yum install tigervnc-server vnc
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@.service
vim /etc/systemd/system/vncserver@.service
 replace USER with the user
systemctl daemon-reload
su - user
vncpasswd
systemctl start vncserver@:display_number.service
systemctl enable vncserver@:display_number.service


# tmux

Basic:
tmux
tmux new •s myname
tmux a  #  (or at, or attach)
tmux a -t myname
tmux ls
tmux kill•session -t myname

Windows:
c  create window
w  list windows
n  next window
p  previous window
f  find window
,  name window
&  kill window

# todo
setting up on an isolated network

using chronyc to control chronyd
 chronyc -h hostname -p port

# tuna
tuna --cpus CPUs --isolate
tuna --irqs IRQs --cpus CPU --move
tuna -q sfc1* -c7 -m -x
tuna --threads thread --priority policy:kernel


# umask

umask shows the default permission values of when a new file is created.

By default, all files are created with 777.

If a umask is 022, the default permission would be 755


# vfat


# x abrt (automatic bug reporting tool)
abrtd daemon

Install (cli or desktop):
 yum install abrt-cli abrt-desktop

Install mail plugin:
 yum install libreport-plugin-mailx

Standard abrt events:
 report_ureport
 report_Mailx
 '_Bugzilla
 '_RHTSupport
 '_EmergencyAnalysis
 '_CCpp
 '_Uploader
 '_VMcore
 '_LocalGDB
 '_xsession_errors
 '_Logger
 '_Kerneloops

To create custom event, they are typically stored in /etc/libreport/events.d/ config files in /etc/libreport/report_event.conf

To setup automatic reporting: abrt-auto-reporting enabled or adding AutoreportingEnabled to /etc/abrt/abrt.conf

Detecting software problems (packages):
abrt-addon-ccpp
abrt-addon-python
rubygem-abrt
abrt-java-connector
abrt-addong-xorg
abrt-addon-kerneloops
abrt-addon-vmcore
abrt-addon-pstoreoops

Command line:
abrt-cli list
abrt-cli info [-d] directory_or_id
abrt-cli report directory_or_id
abrt-cli rm directory_or_id
abrt-cli command --help

# x at
1. Type at TIME
2. Type command at at> prompt and press enter. If want more commands repeat steps.
3. C-D to exit.
to view pending jobs type atq

# x batch
1. Same steps as at but type batch at prompt. 
2. Command is ran when load average decreases < 0.8


# xfs file systems
mkfs.xfs /dev/device
mount /dev/device /mount/point
xfs_growfs /mount/point -D size
xfs_repair /dev/device
xfs_freeze -f /mount/point
xfs_freeze -u /mount/point # unfreeze


# x gparted
check minor-num
cp from to
help
mklabel label
mkfs minor-num file-system-type
mkpart part-type fs-type start-mb end-mb
mkpartfs part-type fs-type start-mb end-mb
move minor-num start-num end-mb
name minor-num name
print
quit
rescue start-mb end-mb
resize minor-num start-mb end-mb
rm minor-num
select device
set minor-num flag state
toggle [NUMBER [FLAG]]
unit UNIT

creating a partition
 parted /dev/sda
 mkpart primary ext3 1024 2048
 mkfs -t ext3 /dev/sda6 # format and label the partition
 e2label /dev/sda6 /work

Make MBR partition table and partition:
   parted /dev/vdb
   mklabel msdos
   mkpart primary 1 1g

# x iptables
```
Parameters:
-A --append
-D --delete
-F --flush
-I --insert
-L --list
-N --new-chain
-R --replace
-X --delete-chain
-d --destination
--dport
-i --in-interface
-j --jump
-m --match
-o --out-interface
-p --protocol
-s --source
-t --table
-v --verbose
```

Some examples:
iptables -F
iptables -t filter -A INPUT -p tcp --dport 80 -j accept # accept port 80
iptables -A OUTPUT -p icmp -j DROP # reject outbound ICMP traffic without sending a notification back
iptables -A FORWARD -d 192.168.0.0./24 -J ACCEPT
iptables -I INPUT -m state --state NEW -p tcp --dport 21 -j ACCEPT # accept the first and subsequent ftp connection
iptables -A OUTPUT -m state --state NEW, ESTABLISHED -p tcp --dport 25 -j DROP # disallow all existing and new outgoing connection requests on port 25
iptables -I INPUT ! -d 192.168.3.3/24 -p ICMP -j DROP # reject all outbound ICMP taffic on all systems on 192.168.3.0/24 except for system with IP 192.168.3.3/24
service iptables save # saves to /etc/sysconfig/iptables

# x isci
```sh
iscsiadm -m session -P 3 # list of sessions
target setup
 vim /etc/tgt/targets.conf
   <target iqn.2008-09.com.example:server.target1>
     backing-store /srv/images/iscsi-share.img
     direct-store /dev/sdd
   </target>
 service tgtd start
 ```

Configure a disk-based iSCSI Target LUN:
```
yum install -y targetcli
systemctl enable target
targetcli
   cd /backstores/block
      create iscsidisk1 dev=/dev/vdb # build a backstore called iscsidisk1
   cd /iscsi
      create iqn.2015-01.com.example.server2:iscsidisk1 # build the iSCSI target with the following address on the iscsidisk1 ackstore
   cd iqn.2015-01.com.example.server2:iscsidisk1/tpg1
      portals/ create 192.168.0.120 # create network portal for target 192.168.0.120 to be used for iSCSI traffic
      luns/ create /backstores/block/iscsidisk1 # create a LUn called lun0 in the target and export it to the network
      set attribute authentication=0 demo_mode_write_protect=0 generate_node_acls=1 # disable authentication, demo_mode_write_protect makes LUN write-enabled andthe generate_node_acls=1 attribute enables the use of TPG-wide authentication settings (disables all ACLS)
   cd /
   exit
   vi /etc/firewalld/servers/iscsitarget.xml
      add <port protocol="tcp" port="3260">
   firewall-cmd --permanent --add-service iscsitarget; firewall-cmd --reload
```
Mount the iSCSI Target on the initiator:
```
yum install -y iscsi-initiator-utils
systemctl enable iscsid
iscsiadm -m discovery -t st -p 192.168.0.120 # use to locate available iSCSI targets from the specific portal (-p)
iscsiadm -m node -T iqn.2015-01.com.example.server2:iscsidisk1 -p 192.168.0.120 -l # login in node mode as specified porta to establish a target/initiator session
isciadm -m session -P3 # view info on established iSCSI session
vi /etc/iscsi/initiatorname.iscsi # add target info on iscsi
   InitiatorName=iqn.2015-01.com.example.server2:iscsidisk1
lsblk | grep sda # grep to see new LUN
fdisk -l | grep sda # grep to see new LUN

Now use LVM to initiate the LUN:
pvcreate -v /dev/sda
vgcreate -v vgsiscsi /dev/sda
lvcreate -L 1G vgisci -n lbiscsi1 -v
mkfs.xfs /dev/vgiscsi/lviscsi1
mkdir /iscsidisk1
vi /etc/fstab
   /dev/vgisci/lvisci1 /iscsidisk1 xfs `_netdev` 0 0
mount /iscsidisk1
df -h | grep iscsidisk1
```



# x todo printing

# x todo sftp

# x virsh
virsh -c lxc:/// start container_name
virsh -c lxc:/// edit container_name
 <domain type='lxc'>
 <emulator>/usr/libexec/libvirt_lxc</emulator>

# ftp

Configure an ftp server:
```
yum -y install vsftpd
mkdir /var/ftp/pub/rhel7
firewall-cmd --permanent --add-service=ftp; firewall-cmd --reload
systemctl enable vsftpd
systemctl start vsftpd
```
# RHCSA OBJECTIVES

Personal notes:
Make sure to go over partitioning / LVM fully. MOST IMPORTANT.
SELinux review.

Go over these objectives:
https://www.redhat.com/en/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam

Understand and use essential tools:
Access a shell prompt and issue commands with correct syntax
Use input•output redirection (>, >>, |, 2>, etc.)
Use grep and regular expressions to analyze text
Access remote systems using ssh
Log in and switch users in multiuser targets
Archive, compress, unpack, and uncompress files using tar, star, gzip, and bzip2
Create and edit text files
XCreate, delete, copy, and move files and directories
Create hard and soft links
List, set, and change standard ugo/rwx permissions
Locate, read, and use system documentation including man, info, and files in /usr/share/doc

Operate running systems:
Boot, reboot, and shut down a system normally
Boot systems into different targets manually
Interrupt the boot process in order to gain access to a system
Identify CPU/memory intensive processes, adjust process priority with renice, and kill processes
Locate and interpret system log files and journals
Access a virtual machine‘s console
Start and stop virtual machines
Start, stop, and check the status of network services
Securely transfer files between systems

Configure local storage:
List, create, delete partitions on MBR and GPT disks
Create and remove physical volumes, assign physical volumes to volume groups, and create and delete logical volumes
Configure systems to mount file systems at boot by Universally Unique ID (UUID) or label
Add new partitions and logical volumes, and swap to a system non•destructively

Create and configure file systems:
Create, mount, unmount, and use vfat, ext4, and xfs file systems
Mount and unmount CIFS and NFS network file systems
Extend existing logical volumes
Create and configure set•GID directories for collaboration
Create and manage Access Control Lists (ACLs)
Diagnose and correct file permission problems

Deploy, configure, and maintain systems:
Configure networking and hostname resolution statically or dynamically
Schedule tasks using at and cron
Start and stop services and configure services to start automatically at boot
Configure systems to boot into a specific target automatically
Install Red Hat Enterprise Linux automatically using Kickstart
Configure a physical machine to host virtual guests
Install Red Hat Enterprise Linux systems as virtual guests
Configure systems to launch virtual machines at boot
Configure network services to start automatically at boot
Configure a system to use time services
Install and update software packages from Red Hat Network, a remote repository, or from the local file system
Update the kernel package appropriately to ensure a bootable system
Modify the system bootloader

Manage users and groups:
Create, delete, and modify local user accounts
Change passwords and adjust password aging for local user accounts
Create, delete, and modify local groups and group memberships
Configure a system to use an existing authentication service for user and group information

Manage security:
Configure firewall settings using firewall•config, firewall-cmd, or iptables
Configure key•based authentication for SSH
Set enforcing and permissive modes for SELinux
List and identify SELinux file and process context
Restore default file contexts
Use boolean settings to modify system SELinux settings
Diagnose and address routine SELinux policy violations

# RHCSA: yum
yum help
yum list
yum search KEYWORD
yum info PACKAGENAME
yum install PACKAGENAME
yum update PACKAGENAME
yum remove PACKAGENAME

Groups of packages:
yum group list
yum group info “Identity Management Server”
yum group install “Infiniband Support”

Enabling Red Hat software repositories:
yum repolist all
yum-config-manager --enable rhel-7-server-debug-rpms

Enabling third-party repos:
yum-config-manager --add-repo=”http://dl.fedoraproject.org/pub/epel/7/x86_64/”


Yum repo:
   vim /etc/yum.conf

# RHCSA: rhel networking

Creating network connetcitons with nmcli:
nmcli con add con-name “default” type ethernet ifname eth0 # DHCP
nmcli con add con-name “static” ifname eth0 autoconnect no type ethernet ipv4 172.25.X.10/24 gw4 172.25.X.254 # Static

Or just use `nm-tui!`

# RHCSA: acl - access control lists

Example file:
ls -l roster.txt
-rwxrw----+ 1 student controller 130 Mar 19 23:56 roster.txt

The "+"  indicates that there are ACL settings associated with this file.

View file ACLs:
getfacl roster.txt
```
# file: roster.txt
# owner: student
# group: controller
user::rwx
user:james:---
user:1005:rwx     #effective:rw-
group::rwx        #effective:rw-
group:sodor:r--
group:2210:rwx    #effective:rw-
mask::rw-
other::---
```

setfacl -m acl_spec # to add or modify
setfacl -x acl_spec # to delete

d:. include execute permissions to sensure access to new subdirectories
-R for recursive, -b to delete all ACLs, -k to delete all default ACLs

Add or modify a group or named group ACL:
setfacl -m g:name:rw file

Add or modiy OTHER ACL:
setfacl -m o::- file

Setting an explicit ACL mask:
A mask can be explicitley set on a file to limit the maximum effective permissions for named users, group-owner and groups.

Determine, set and delete ACLs:
pwd
touch file1
getfacl file1
mount -o remount,acl /home (edit /etc/fstab and replace defaults with acl)
setfacl -m u:user3:7 file7 # adds read/write/execute to user3
setfacl -m u:user4:rw file1 # same with rw
setfacl -x u:user3 file1 # deletes acl for user3 on file

Set, confirm and delete default ACLs:
setfacl -m d:u:user1:6,d:u:user3:6 projects # allocate default read and write permissions for user1 and user3 (the d part)
setfacl -k project # deletes all default ACL settings

Deny the user james from sodor group any access:
setfacl -Rm u:james:- /shares/steamies

Set to recursively update steamies dir, grating sodor group read, write and conditional execute permissions:
setfacl -Rm g:sodor:rwX /shares/stemies

Determining that ACL is enabled:
mount | grep vda
/dev/vda1 on / type ext4 (rw,relatime,seclabel,data=ordered)
hint: it’s not
vim /etc/fstab
   UUID=b6c9ab51-aae3-4c1d-a941-2a3a6d55a925 / ext4 defaults,acl 1 1

ACL configuration:
setfacl -m u:bob:rw- f # allow read/write to file f
setfacl -x u:bob f # remove permissions allowed to user bob
setfacl -b f # remove ALL acl’s on file
setfacl -R  -m g:team:r-x dir # allow read/execute permissions to team on directory dir recursively


# RHCSA: authconfig - kerberos authentication
Kerberos authentication

To configure a system for LDAP + Kerberos:
yum install authconfig-gtk sssd krb5-workstation

Within the GTK:
LDAP Search Base DN: dc=example,dc=com
LDAP Server: classroom.example.com
Use TLS
Kerberos password
Realm: EXAMPLE.COM
KDCs: classroom.example.com
Admin Servers: classroom.example.com

To test, simple SSH to the server (should be able to access / login).

# RHCSA: chage

Get a date in the future and expire on that date:
date -d “+180 days”
chage -E 2014-08-02 romeo

# RHCSA: chmod

0 | 000 | --- | no permissions
1 | -01 | --x | execute 
2 | 010 | -w- | write
3 | 011 | -wx | write and execute
4 | 100 | r-- | read
5 | 101 | r-x | read and execute
6 | 110 | rw- | read and write
7 | 111 | rwx | read, write and execute

First number - Read, write, execute for user
Second number - Read, write, execute for group
Third number - Read, write, execute for other

chmod WhoWhatWhich file|directory
Who is u, g, o, a (user, group, other, all)
What is +, -, = (add, remove set)
Which is r, w, x (read, write, execute)

Setuid:
File will run as the user (or group) of the file, not the user that ran the command.

Setgid:
setting setgid bit. allows files and folders created in that directory to inherit the dir’s owning group

Stickybit:
The stickybit sets a special restriction on delection of files. Only the owner of the file (and root) can delete files within the dir.

Number | Special permission | Effect on files                               | Effect on dirs
------------------------------------------------------------------------------------------------
4      | u+s (suid)         | Executes as the user that owns the file.      | No effect
2      | g+s (sgid)         | File executes as the group that owns the file | New files have group owner of the dir
1      | o+t (sticky)       | No effect                                     | Users with write on dir can only remove files they own

Remove read and write permission for group and other on file1:
chmod go-rw file1

Add execute permission on file2:
chmod a+x file2

Add the setgid bit on dir:
chmod g+s directory

Set the setgid bit and read/write/execute for user and group of directory:
chmod 2770 directory

Example:
groupadd -g 50000 team
mkdir /home/shared
chown nobody:team /home/shared
chmod g+s /home/shared
chmod g+w /home/shared
chmod o-wrx /home/shared
useradd -G team user1
useradd -G team user2
chmod +t /home/shared # if you want members to see files but not delete

# RHCSA: chown

Grant ownwership of foofile to student:
chown student foofile

Change group ownership of a file:
chown :admins foodir

# RHCSA: fdisk - creating MBR partitions

Creating an MBR disk partition:
fdisk /dev/vdb

and follow the instructions...

Make sure to partprobe /dev/vdb after

# RHCSA: gdisk - creating GPT partitions

Creating a GPT disk:

gdisk /dev/vdb

and follow instructions..

Make sure to partprobe /dev/vdb after

# RHCSA: gnome (graphical installation)

RHEL:

First, install:
yum group list
yum groupinstall ‘Server with GUI’

Then change the runlevel to graphical:
systemctl enable graphical.target --force

# RHCSA: groupadd
[options] group_name

Creating group directories:
mkdir /opt/myproject
groupadd myproject
chown root:myproject /opt/myproject
chmod 2775 /opt/myproject

Create a group with a gid of 5000:
groupadd -g 5000 linuxadm

Create another group sharing the same gid:
groupadd -o -g 5000 sales

Change the gid of group linuxadm:
groupmod -g 6000 linuxadm

Add a user to group linuxadm:
usermod -a -G linuxadm user

Change user to primary group student:
usermod -g student student

Change shell of the user:
usermod -s /sbin/nologin student
usermod -s /usr/bin/zsh student

# RHCSA: hostname
hostnamectl set-hostname linux.domain.com
vi /etc/hostname

# RHCSA: kill
kill PID
kill -signal PID
kill -l # list of signals
killall -signal -u username command_pattern


# RHCSA: ldap
Configure LDAP client to obtain user and group info:
server2 has openldap service such as freeipa or red hat directory server configured and running and has user account ldapuser1 and group dba available to test auth.

yum -y install openldap openldap-clients nss-pam-ldapd sssd authconfig
authconfig —enableldap —enableldapauth —ldapserver=ldap://server2.example.com —enablesssd —ldapbasedn”dc=example,dc=com” —update
vim /etc/nssswitch.conf # ensure passwd, gshadow and group entires look like:
   passwd: files sss
   shadow: files sss
   group: files sss

Note: use authconfig-tui for ldap client setup with or without Kerberos !!

Client configuration:
```sh
yum install -y openldap-clients nss-pam-ldapd
yum group install "Directory Client" # alternative for downloading
authconfig --enableforcelegacy --update
authconfig --enableldap --enableldapauth --ldapserver="instructor.example.com" \
--ldapbasedn="dc=example,dc=com" --update
```

Pull the certification over:
```sh
scp root@instructor.example.com:/etc/openldap/certs/cert.pem /etc/openldap/cacerts/cert.pem
restorecon /etc/openldap/cacerts/cert.pem # apply correct selinux permissions
authconfig --enableldaptls --update
```

# RHCSA: linux kernel

Upgrading kernel:
RHEL is a custom-built kernel by the RHEL kernel team to ensure integrity and compatability with supported hardware. Kernels are packaged in the RPM format and are verifyable / upgradable using Yum or PackageKit package managers.

To determine which kernel pakcages are installed use: 
yum list installed "kernel-*"

# RHCSA: linux logs
Usually managed by either rsyslogd or journald (component of systemd)
Logs located in /var/log
to edit logging edit /etc/logrotate.conf

Log all messages with debug to /var/log/messages-debug:
echo “*.debug /var/log/messages-debug” > /etc/rsyslog.d/debug.conf
systemctl restart rsyslog

Store the system journal permanently:
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/journal
systemctl reboot or killall -USR1 systemd-journald

Commands:
journalctl -p priority -o form -n number -f --since=value --until=value

Advanced filtering:
journalctl -F fieldname
journalctl fieldname=value fieldname1=value1

Persistent storage: To enable persis storage simply:
mkdir -p /var/log/journal
systemctl restart systemd-journald

Graphical environment journal viewing:
gnome-system-log

# RHCSA: localectl
Setting a locale:
localectl list-locales
localectl set-locale LANG=locale

Setting a keymap:
localectl list-keymaps
localectl set-keymap map
localectl set-x11-keymap map

# RHCSA: loghost
Configure system as a loghost:

vi /etc/rsyslog.conf # uncomment these lines:
   $ModLoad imtcp
   $InputTCpServerRun 514
firewall-cmd --permanent --add-port 514/tcp
firewall-cmd --reload
semanage port -a -t syslogd_port_t -p tcp 514
systemctl enable rsyslog
systemctl restart rsyslog

Configure system as a loghost client:

vi /etc/rsyslog.conf # add following to bottom
`*.* @@192.168.0.120:514`
systemctl enable rsyslog
systemctl restart rsyslog

# RHCSA: crontab
Crontab represented by:
minute hour day month dayofweek username command
* to specify all values
- for in between integers
, list
/ specify step values

20 1,12 1-15 * * find / -name core -exec rm {} \:

field | content
———————————————
1     | minute of hour | 0 to 59
2     | hour of day | 0 to 23
3     | day of month |  1 to 31
4     | month of the year | 1 to 12
5     | day of the week | 0 to 6, 0 = sunday, 1 = monday, etc.
6     | command or script to execute | full path name of the command or script to be executed

crontab -e # edit
crontab -l # list

Daily count job on number of active users:
vim /etc/cron.daily/usercount
   #!/bin/bash
   USERCOUNT=$(wc -h | wc -l)
   logger “There are currently $(USERCOUNT) active users”
chmod +x /etc/cron.daily/usercount

Installing sysstat and change cron.daily to 5 minutes:
yum -y install sysstat
rpm -qc sysstat # view configuration files to find what cron jobs are used
vim /etc/cron.d/sysstat
   change */10 sa1 line to */5

5th min past the hour from 1am-5am on the 1st and 15th of alternative months:
```
*/5 1-5 1,15 */2 * /home/user100/script100.sh &> /tmp/script100.out
```

# RHCSA: tmp files cleaning

Changing to cleaning every 5 days:
sudo cp /usr/lib/tmpfiles.d/tmp.conf /etc/tmpfiles.d/
vim /etc/tmpfiles.d/tmp.conf # change to 5d
sudo systemd-tmpfiles --clean tmp.conf # test cleaning

Create a new file and clear /run/gallifrey dir:
vim /etc/tmpfiles.d/gallifrey.conf
   d /run/gallifrey 0700 root root 30s
sudo systemd-tmpfiles —create gallifrey.conf # Test the new configuration

# RHCSA: /etc/fstab
Anything defined in fstab are mounted automatically at reboots
beauty of mount command / fstab is it will automatically detect what filesystem the distro is using and use that accordingly, whether it is xfs, ext4, ext3, etcetera.

Example:
/dev/mapper/vg00-root / xfs defaults 1 1
UUID=01fab63f-7902-4f3c-b48c-7db538e96562 / ext4 errors=remount-ro 0 1
/dev/mapper/vg00-swap swap swap defaults 0 0

Options:
uuid, mount point, type, mount options, 0 = whether to dump or not [bool], 1 = boot order [0-9]

If any missing or invalid information renders the system unbootable. Have to boot into single-user mode.

/dev/mapper/vg00-root         /          xfs          defaults         1             1
Mapping point / UID         Mount        Type         Options     dump/no-dump   fsck. 0 = no check, 1 = root, 2 = after root



# RHCSA: file systems - lvm

Creating a logical volume:
fdisk /dev/vdb # create a new partition
pvcreate /dev/vdb1 /dev/vdb2 # creates the physical volumes
vgcreate shazam /dev/vdb1 /dev/vdb2 # creates the volume group built from the two PVs
lvcreate -n storage -L 400M shazam # create a logical volume of 400MiB
mkfs -t xfs /dev/shazam/storage # makes it xfs file system on the storage LV
mkdir /storage
vim /etc/fstab
   /dev/shazam/storage /storage xfs defaults 1 2
mount -a # mount it / test
fdisk -l /dev/vdb # view all entries

Extending a volume:
vgdisplay shazam
fdisk /dev/vdb # create a new partition of 512MiB
pvcreate /dev/vdb3
vgextend shazam /dev/vdb3
lvextend -L 700M /dev/shazam/storage # extend existing LV to 700MiB
xfs_growfs /storage # extend the XFS file system to the remainder of the free space

Extending a volume to 100%:
```
mkfs.ext4 /dev/vg/lv_vol
mount /dev/vg/lv_vol /mnt
lvextend -l +100%FREE -r /dev/vg/lv_vol # extend to 100%
lvextend --size +50M -r /dev/vg/lv_col
```

Reducing a file system:
```
umount /dev/vg/lv_vol
lvresize --size -50M -r /dev/vg/lv_vol
mount /dev/vg/lv_vol /mnt
```

XFS extending within LVM:
```
mkfs.xfs /dev/vg/lv_vol
mount /dev/vg/lv_vol /mount
lvextend --size +50M -r /dev/vg/lv_vol
xfs_growfs /mnt # if you DONT do -r on lvextend
```


Resizing a system:
lvrename vg01 lvol0 lvolnew
lvs | grep lvvolnew
lvreduce -L 800m /dev/vg01/lvol0
lvresize -L 700m /dev/vg01/lvolnew # lvresize or lvreduce, doesn’t matter

Removing them:
lvremove -f /dev/vg01/lvolnew
lvremove -f /dev/vg01/oravol

Creating swap space:
mkswap /dev/vdb2
lvcreate -L 300m -n swapvol vg10
mkswap /dev/vg10/swapvol
vim /etc/fstab
   /dev/vg10/swapvol swap swap defaults 0 0
   UUID=blahblah     swap swap defaults 0 0

# RHCSA: file systems - xfs

Making a xfs fs:
mkfs -t xfs /dev/vdb1
vim /etc/fstab
   UUID=7a20315d-ed8b-4e75-a5b6-24ff9e1f9838 / xfs defaults 1 1

Create a 2 GiB XFS on GTP mounted at /backup. 512MiB swap partition on second disk ith default. Another 512 MiB swap with priority of 1:
gdisk /dev/vdb # make the disks
partprobe
mkfs -t xfs /dev/vdb1
mkswap /dev/vdb2
mkswap /dev/vdb3
mkdir /backup
blkid /dev/vdb1
vim /etc/fstab
UUID=748ca35a-1668-4a2f-bfba-51ebe550f6f0 /backup xfs defaults 0 2
UUID=d00554b7-dfac-4034-bdd1-37b896023f2c swap swap defaults 0 0
UUID=af30cbb0-3866-466a-825a-58889a49ef33 swap swap pri=1 0 0

XFS - Create, mount and extend an xfs file system:
pvcreate /dev/vdc1
vgextend vg10 /dev/vdc1
lvcreate -L 188m -n lvolxfs vg10 /dev/vdc1
mkfs.xfs /dev/vg10/lvolxfs
mkdir /mntxfs
mount /dev/vg10/lvolxfs /mntxfs
vi /etc/fstab
   /dev/vg10/lvolxfs /mntxfs xfs defaults 1 2
lvresize -r -L 300m /dev/vg10/lvolxfs
reboot

XFS + LVM - Create mount, unmount and remove file systems:
parted /dev/vdc mkpart primary 202 303m
mkfs.xfs /dev/vdc2
pvcreate /dev/vde2
vgcreate vg20 /dev/vde2
lvcreate -L 96m -n lvolext4rem vg20
mkfs.ext4 /dev/vg20/lvolext4rem
mkdir /mntxfsrem /mntext4rem
xfs_admin -L mntxfsrem /dev/vdc2 # apply label
mount LABEL=mntxfsrem /mntxfsrem
mount /dev/vg20/lvolext4rem /mntext4rem
vi /etc/fstab
   LABEL=mntxfsrem /mntxfsrem xfs defaults 1 2
   /dev/vg20/lvolext4rem /mnt/ext4rem ext4 defaults 1 2
enmount /mntxfsrem
fuser -cu /mntxfsrem (check to see what processes are using this mount point)
unmount /mnt/mntxfsrem /mntext4rem
parted /dev/vdc1 rm 2
lvremove -f /dev/vg20/lvolext4rem
vgremove vg20
rmdir /mntxfsrem /mntext4rem

XFS - Creating volume:
mkfs.xfs /dev/vg/lv_vol
vim /etc/fstab
   /dev/vg/lv_vol /mnt xfs defaults 1 2

# RHCSA: file systems - ext4

Adding a partition, file system and a persistent mount:
fdisk /dev/vdb # create the partition
partprobe # detect it
mkfs -t ext4 /dev/vdb1
mkdir /archive
blkid /dev/vdb1 # to find out the UUID
vim /etc/fstab
UUID=5fcb234a-cf18-4d0d-96ab-66a4d1ad08f5 /archive ext4 defaults 0 2
mount -a # to mount any new entries that have been added to /etc/fstab

EXT4 - Create and mount an extended file system:
parted /dev/vdb mklabel msdos
parted /dev/vdb mkpart primary ext3 1 201m
mke2fs -t ext3 /dev/vdb1
pvcreate /dev/vdd -v
vgcreate -v vg10 /dev/vdd
lvcreate -L 1.5g -n lvolext4 vg10 -v
mke2fs -t ext4 /dev/vg10/lvolext4
vi /etc/fstab
   UUID=c8dd716...    /mntext3 ext3 defaults 1 2
   /dev/vg10/lvolext4 /mntext4 ext4 defaults 1 2

EXT4 - Resize an extended file system:
parted /dev/vdb mkpart primary 202m 703m
pvcreate /dev/vdb2
vgextend vg10 /dev/vdb2
lvresize -r -L 2g /dev/vg10/lvolext4
lvresize -r -L 1.1g /dev/vg10/lvolext4

EXT4 - Create from a LVM volume
```
lvcreate --size 100M --name lv_vol /dev/vg
mkfs.ext4 /dev/vg/lv_vol
mount /dev/vg/lv_vol /mnt
vim /etc/fstab
   /dev/vg/lv_vol /mnt ext4 defaults 1 2
```

# RHCSA: file systems - swap

Making swap:
fdisk /dev/vdb
mkswap /dev/vdb1
swapon -a # activates all swap spaces listed in /etc/fstab
vim /etc/fstab
UUID=fbd7fa60-b781-44a8-961b-37ac3ef572bf swap swap defaults 0 0

# RHCSA: file systems - vfat

VFAT - Create a vfat:
parted /dev/vde mklabel msdos
parted /dev/vde mkpart primary fat32 1 401m
mkfs.vfat /dev/vde1
mkdir /mntvfat
mount /dev/vde1 /mntvfat
blkid /dev/vde1
vi /etc/fstab
   UUID=0183209123 /mntvfat vfat defaults 1 2

# RHCSA: file systems - NFS

Mounting an NFS dir (ez way):
yum install -y nfs-utils
systemctl enable nfs-idmap && systemctl start nfs-idmap
systemctl enable nfs-client.target && systemctl start nfs-client.target
vim /etc/fstab
   nfsserver:/home/tools /mnt nfs4 defaults 0 0

Mounting an external NFS directory with kerberos:
sudo wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/desktopX.keytab
systemctl enable nfs-secure
systemctl start nfs-secure
sudo mkdir -p /mnt/plubic /mnt/manual
sudo vim /etc/fstab
   serverX:/shares/public /mnt/plubic nfs sec=krb5p,sync 0 0
sudo mount -a # to test
sudo mount -o sync,sec=sys serverX:/shares/manual /mnt/manual # to manually mount

Mounting an NFS share with autofs:
yum -y install autofs
vim /etc/auto.master.d/home.autfs
   /home/guests /etc/auto.home
vim /etc/auto.home
   *  -rw,sync classroom.example.com:/home/guests/&
systemctl enable autofs
systemctl start autofs

NFS automounter client configuration:
```sh
yum install -y autofs nfs-utils
vim /etc/auto.guests
   * -rw,nfs4 instructor.example.com:/home/guests/&
vim /etc/auto.master
   /home/guests /etc/auto.guests
systemctl enable autofs && systemctl start autofs
su - ldapuser02 # to test
```

# RHCSA: file systems - cifs/smb

Mount and unmount CIFS file system:
yum -y install samba-client cifs-utils
`smbclient://192.168.0.110/smbrhcsa -U user1`
`mount //192.168.0.110/smbrhcsa /smbrhscamnt -o username=user1`
vi /etc/samba/smbrhcsacred
   username=user1
   password=user123
vi /etc/fstab
   //192.168.0.110/smbrhcsa /smbrhcsamnt cifs rw,credentials=/etc/samba/smbrhcsacred 0 0

Mounting an CIFS fs alternative example:
yum install -y samba-client cifs-utils
vim /etc/fstab
   //smbserver/shared /mnt cifs rw,username=user01,password=pass 0 0
mount -a

Mounting the SMB share:
smbclient -L //serverX # identifies what files are shareable
mkdir -p /mountpoint
mount -t cifs -o guest //serverX/share /important # or
vim /etc/fstab
   //serverX/share /mountpoint cifs guest 0 0

Mounting automatically with the automounter:
vim /etc/auto.msater.d/bakerst.autofs
   /bakerst /etc/auto.bakerst
vim /etc/auto.bakerst
   cases -fstype=cifs,credentials=/secure/sherlock ://serverX/cases
vim /secure/sherlock # owned by root, perms 600
   username=sherlock
   password=violin221B
   domain=BAKERST
systemctl enable autofs
systemctl start autofs

Mounting a Samba share:
yum -y install cifs-utils
mkdir ~/work
mkdir /secure
vim /secure/student.smb
   username=student
   password=student
   domain=MYGROUP
chmod 770 /secure
chmod 600 /secure/student.smb
vim /etc/fstab
   //serverX/student /home/student/work cifs credentials=/secure/student.smb 0 0
mount -a # test mounting

Mounting a SMB share (again):
yum install -y cifs-tuils autofs
vim /etc/auto.master.d/shares.autofs
   /shares /etc/auto.shares
vim /etc/auto.shares
   work -fstype=cifs,credentials=/etc/me.cred ://serverX/student
   docs -fstype=cifs,guest ://serverX/public
   cases -fstype=cifs,credentials=/etc/me.cred :/serverX/bakerst
vim /etc/me.cred
   username=student
   password=student
   domain=MYGROUP
chmod 600 /etc/me.cred
groups # to check the curent group memberships for the student
groupadd -g 10221 bakerst
usermod -aG bakerst student
newgrp bakerst # to switch to the new group
systemctl enable autofs
systemctl start autofs

# RHCSA: ipa-client - client to an IPA server

By default, ipa-client can discover and configure itself if the server has been setup correctly:
ipa-client-install

# RHCSA: nice
Range from -20 (most fav) to 19 (least fav)

Unprivileged are allowed to set a positive nice level (0 to 19). Only root can set a negative (-20 to -1).

Start a command with a nice level of 15:
nice -n 15 dogecoinminer &

Changing the nice level of an existing process
renice -n 5 <PID>
renice -n -7 $(pgrep origami@home)

# RHCSA: mount
mount options:
   aysnc - allows file system i/o to have asynch access
   acl
   atime - updates inode access time for each access
   auto - automounts
   defaults - accepts all default values (async, auto, dev, exec, nouser, rw, and suid)
   dev - interpets the files on the filesystem
   exec - permits execution of a ibinary file
   loop - mount iso image as a loop device
   owner - allows file system owner to mount file system
   `_netdev` - used for file system that requires network connectivity before it’s mounted (iscsi, nfs, cifs)
   remount - remounts already mounted file system to enable or deisalbe option
   ro - read only
   rw - write only
   suid - enables running setuid and setgid programs
   user - allows normal user to mount a file system
   users - allows all users to mount and unmount  afile system

Find out new paritions by using blkid:
blkid
/dev/sda1: UUID=“9aeef974-86d1-4fa8-97ff-1613b34521e4” TYPE=”ext4” PARTUUID=”6c230a4a-01”
/dev/sda5: UUID=“7e4d37c8-77bd-422a-816b-4c7427fadaeb” TYPE=”swap” PARTUUID=”6c230a4a-05”

Mounting by device file of the partition:
mount /dev/vdb1 /mnt/mydata

Mount a DVD:
mount -o loop disk1.iso /mnt
or
mount -o loop /dev/sr0 /mnt

UUID:
mount UUID=“46f543fd-78c9-4526-a857-244811be2d88” /mnt/mydata


# RHCSA: RPM
 rpm -Uvh package.rpm
 rpm -Uvh --nodeps --force package.rpm
 rpm -qi package
 rpm -qip package.rpm
 rpm -e package
 rpm -e --nodeps package
 rpm -qa |grep package
 rpm -q package --qf %{name}-%{version}%{arch}
 rpm --showrc
 rpm -q package --requires
 rpm -q package --provides
 rpm -q package --whatrequires
 rpm --root /some/path -Uvh package.rpm 
 rpm -q package --scripts # 
 rpm --rebuilddb # Rebuild rpm database

# RHCSA: ssh
Files:
 /etc/ssh/moduli - Diffie-Hellman groups for key exchange
 /etc/ssh/ssh_config - default SSH client config file
 /etc/ssh/sshd_config - default daemon config file
 ../ssh_host_ecdsa_key
 ../ssh_host_ecdsa_key.pub
 ../ssh_host_key
 ../ssh_host_key.pub
 ../ssh_host_rsa_key
 ../ssh_host_rsa_key.pub
 /etc/pam/.d/sshd - PAM config file for sshd daemon
 /etc/sysconfig/sshd - config file for the sshd service

User files:
 ~/.ssh/authorized_keys
 ../id_ecdsa
 ../id_ecdsa.pub
 ../id_rsa
 ../id_rsa.pub
 ../identity
 ../identity.pub
 ../known_hosts

Enabling the SSH daemon:
systemctl start sshd.service
systemctl stop sshd.service
systemctl enable sshd.service

Port forwarding:
ssh -L port:hostname:remote-port username@localhost
ssh -L 1100:mail.example.com:110 mail.example.com

Types of authentication:
GSSAPI-Based authentication - Kerberos to be plugged in
Host-Based authentication - single user or grou pof users to authenticate
Private/Public Key-Based authentication - uses private/public key combination for user authentication
Challenge-response authentication - responses to arbitrary challenge questions that the user has to answer correctly
Password-based authentication - last fall-back option, server prompts user to enter pass and checks stored entry in the shadow file

Copying an SSH key to a different host:
ssh-copy-id -i ~/.ssh/id_rsa.pub root@server.example.com

# RHCSA: sudo

Give full admin privileges on sudo:
usermod -G wheel <username>
visudo
   juan ALL=(ALL) ALL

# RHCSA: systemd
Unit types:
.service
.target
.automount
.device
.mount
.path
.scope
.slice
.snapshot
.socker
.swap
.timer

Systemd locations:
/usr/lib/systemd/system
/run/systemd/system
/etc/systemd/system

Commands:
stop bluetooth
stop bluetooth.service (can omit .service if it's a service already)
start name.service
stop name.service
restart name.service
try-restart name.service
reload name.service
status name.service
is-active name.service
list-units --type service --all
enable name.service
disable name.service
status name.service
is-enabled name.service
list-unit-files --type service

Targets:
in systemd, run levels in sysV are replaced by target levels in systemd
   0. poweroff.target
   1. rescue.target
   2. multi-user.target
   3. graphical.target
   4. reboot.target
commands
systemctl 
   list-units --type target --all
   isolate name.target
   set-default name.target

Setting a default target:
systemctl set-default graphical.target

Rescue:
 systemctl rescue # reboot to rescue mode.
 systemctl emergency # emergency boot

Power commands:
systemctl 
 halt
 poweroff
 reboot
 suspend
 hibernate
 hybrid-sleep #hibernates and suspends the system

Controlling systemd via ssh:
systemctl --host user@host command

Docker systemd file:
Automatically starting a container on boot
 [Unit]
 Description=Redis container
 Author=Me
 After=docker.service

 [Service]
 Restart=always
 ExecStart=/usr/bin/docker start -a redis_server
 ExecStop=/usr/bin/docker stop -t 2 redis_server

 [Install]
 WantedBy=local.target

Booting to emergency to fix /etc/fstab:
edit GRUB entry at boot, append: systemd.unit=emergency.target at the end of the line
Ctrl+X
mount -o remount,rw /
mount -a # comes up with mounting error
vim /etc/fstab # remove invalid line
mount -a
reboot

Generating a new  grub install:
grub2-mkconfig > /boot/grub2/grub.cfg
systemctl reboot

# RHCSA: tar
Creating a tarball:
tar cvf /tmp/home.tar stuff

Append files:
tar rvf /tmp/home.tar /etc/yum.repos.d

List:
tar tvf /tmp/files.tar

Restore /home from home.tar:
tar xvf /tmp/home.tar

Extract files from files.tar to /tmp dir:
cd /tmp
tar xvf /tmp/files.tar

# RHCSA: grub2
Grub2 reads its config from /boot/grub2/grub.cfg on traditional BIOS based machines and /boot/efi/EFI/redhat/grub.cfg on UEFI machines
Grub is generated by using the /usr/sbin/grub2-mkconfig utility
Ex.
 grub2-mkconfig -o /boot/grub2/grub.cfg
 grub2-mkconfig -o /boot/ufi/EFI/redhat/grub.cfg
Within grub.cfg are menuentries for Linux. Each one represents an installed Linux kernel containing linux (linuxefi on UEFI systems) and initrd directives followed by the path to the kernel and the initramfs image.
Kernel version number as given on the linux /vmlinuz-kernel_version line must match the version number of the initramfs image given on the initrd /initramfs-kernel_version.img line of each menuentry block.

Grub uses a series of scripts to build the menu; these are located in the /etc/grub.d/ directory:
 00_header, loads GRUB2 settings from /etc/default/grub
 01_users, created only when boot loader pass assigned in kickstart
 10_linux, locates kernels in the default partition of RHEL
 30_os-prober, builds entries for OS found on other partitions
 40_custom, template which can be used to create additional menu entries

By default, the saved value is used for the GRUB_DEFAULT key in the /etc/default/grub file. Instructs GRUB2 to load the kernel specified by the saved_entry directive in the GRUB2 environment file, located at /boot/grub2/grubenv. You can set another GRUB record to be the default using the "grub2-set-default" command which will update the GRUB2 environment file. Ex: grub2-set-default 2

Remember, changes to /etc/default/grub require rebuilding the grub.cfg (grub2-mkconfig -o /path/) ex: grub2-mkconfig -o /boot/grub2/grub.cfg

To run in emergency mode:
Press e key to edit the kernel parameters on boot. Add 'emergency' as a paramter to the end of the menu line. To make it persistent, edit the values of the GRUB_CMDLINE_LINUX key in the /etc/default/grub file. Ex. GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,9600n8"

Installing/reinstalling grub2:
grub2-install /dev/sda

To use with serial, edit /etc/default/grub:
GRUB_TERMINAL="serial"
GRUB_SERIAL_COMMAND="serial --speed=9600 --unit=0 --word=8 --parity=no --stop=1"
use screen to connect: sceen /dev/<console_port>

Rescue mode:
add systemd.unit=rescue.target to the end of the linux16 or linuxefi boot menu entry.

Emergency:
add systemd.unit=emergency.target to the end of the linux16 or linuxefi boot menu entry.

Resetting the root password the Red Hat book way:
Interrupt the system, edit entry on boot
add rd.break 
ctrl+x to boot with changes
mount -o remount,rw /sysroot # remount sysroot as read-write
chroot /sysroot # switch to a chroot jail
passwd root
touch /.autorelabel
exit twice

# RHCSA: timedatectl

Setting a time:
timedatectl set-time YYYY-MM-DD
timedatectl set-time HH:MM:SS
timedatectl set-local-rtc boolean

Setting a timezone:
timedatectl list-timezones
timedatectl set-timezone time_zone

Setting ntp:
timedatectl set-ntp yes

Getting X time in the future:
date +%F -s YYYY-MM-DD
date +%T -s HH:MM:SS

# RHCSA: chrony

chronyd is a daemon running in userspace and an alternative to ntp. chronyc is the command-line program to interact with chronyd

Default config is  in /etc/chrony.conf:
allow
cmdallow
dumpdir
dumponexit
local
log
logdir
makestep 1000 10
maxchange 1000 1 2
maxupdatesskew
noclientlog
reselectdist
stratumweight
rtcfile
rtcsync

To add servers to sync to add to /etc/chrony.keys:
server w.x.y.z key 10
peer w.x.y.z key 10

Installing/user:
yum install chrony
yum install chrony -y
systemctl status chronyd
systemctl start chronyd
systemctl enable chronyd
chronyc tracking - tracking
chronyc sources
chronyc sourcestats
chronyc -a makestep - change time

Setting up chrony on an env that is infrequently changed:
edit /etc/chronyd
   driftfile /var/lib/chrony/drift
   commandkey 1
   keyfile /etc/chrony.keys
add ntp servers to /etc/chrony.keys
   server 0.pool.ntp.org offline
   server 1.pool.ntp.org offline
   server 2.pool.ntp.org offline
   server 3.pool.ntp.org offline

Setting up a correct time zone for serverX:
tzselect
timedatectl set-timezone America/Port-au-Prince
vim /etc/chrony.conf
   server classroom.example.com iburst
systemctl restart chronyd
timedatectl set-ntp true

# RHCSA: firewalld

Default configuration of firewalld zones:
trusted - allow all incoming
home/internal - reject incoming unless related to ssh, ipp-client, samba-client or dhcpv6-client
work - reject incoming unless matching ssh, ipp-client or dhcpv6-client
public - reject incoming unless matching ssh or dhcpv6-client
external - reject incoming unless matching ssh, outgoing traffic is masqueraded to look like it originated from the IPv4 address of the outgoing network interface
dmz - reject incoming unless matching ssh
block - reject all incoming
drop - drop all incoming

Default pre-defined firewalld services:
ssh
dhcpv6-client
ipp-client - local ipp printing
samba-client
mdns - multicast DNS local-link name resolution (port 5353)

Parameters:
```
--state
--reload
--permanent
--get-default-zone
--get-services
--list-all
--list-services
--add-service
--remove-service
--query-service
--list-ports
--ad-port
--remove-port
--query-port
--list-forward-ports
--ad-foward-port
--remove-forward-port
--query-foward-port
--list-interfaces
--add-interfaces
--remove-interfaces
--query-interfaces
```

To install/use:
systemctl stop iptables
systemctl disable iptables
systemctl enable firewalld
systemctl start firewalld

All incoming from 192.168.0.0/24 open for internal and adding mysql:
firewall-cmd --set-default=dmz
firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24
firewall-cmd --permanent --zone=internal --add-service=mysql
firewall-cmd --reload

Running an http server on 80 and 443:
yum install -y httpd mod_ssl
echo “I am alive” > /var/www/html/index.html
systemctl start httpd
systemctl enable httpd
systemctl mask iptables
systemctl mask ip6tables
systemctl status firewalld firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --reload

Setting 8080 to public:
systemctl mask iptables
systemctl mask ip6tables
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --get-default-zone
firewall-cmd --set-default-zone public
firewall-cmd --permanent --zone=public --list-all # check
firewall-cmd --permanent --zone=public --add-port=8080/tcp
firewall-cmd --permanent --zone=public --list-all # check
firewall-cmd --reload


Add and manage firewall rules:

```
firewall-cmd --get-default-zone
firewall-cmd --permanent --add-service=http # adds perm rule to allow http traffic
firewall-cmd --reload # actives rule
firewall-cmd --add-port=443/tcp # allow traffic on tcp port 443
firewall-cmd --permanent --add-port=5901-5910/tcp; firewall-cmd --reload
```

Find and use firewall zones:
firewall-cmd --get-default-zone
firewall-cmd --get-active-zones
firewall-cmd --new-zone testzone --permanent
cat /etc/firewalld/zones/testzone.xml # file for firewalld zone testzone
firewall-cmd --delete-zone testzone --permanent
firewall-cmd --set-default-zone external

View and manage services:
firewall-cmd --permanent --new-service testservice
vi /etc/firewalld/services/testservice.xml
   firewall-cmd --permanent --add-service testservice --zone work
   firewall-cmd --reload
   firewall-cmd --list-services --zone work

Add ports and stuffs:
firewall-cmd --list-ports
firewall-cmd --permanent --add-port 53/tcp
firewall-cmd --add-port 1000-1010/udp --zone work
firewall-cmd --permanent --remove-port 53/tcp; firewall-cmd --reload

Manage using rich language:
firewall-cmd --add-rich-rule 'rule family="ipv4" source address="192.168.3.0/24" \
service name="http" log prefix="HTTP Allow Rule" level="info" accept' --permanent
cat /etc/firewalld/zones/public.xml
firewall-cmd --add-rich-rule 'rule family="ipv4" source address="192.168.4.0/24" \
service name="telnet" log prefix="telnet Access Denied" level="info" reject' \
--timeout="86400" --zone dmz
firewall-cmd --list-rich-rules
firewall-cmd --list-rich-rules --zone dmz

Add and remove masquerading:
firewall-cmd --zone external --ad-masquerade
firewall-cmd --query-masquerade --zone external
firewall-cmd --remove masquerade --zone external

Add and remove port forwarding:
firewall-cmd --zone external --add-masquerade
firewall-cmd --zone external --add-forwarding-port port=23:proto=tcp:toport=1000 --permanent
firewall-cmd --zone external --permanent --add-forward-port port=21:proto=tcp:toport=1001-1005
firewall-cmd --zone external --permanent --add-forward-port port=69:proto=tcp:toport=1010:toaddr=192.168.0.121
firewall-cmd --reload

Adding an IP set:
```sh
firewall-cmd  --permanent --new-ipset=blacklist --type=hash:ip
firewall-cmd --reload
firewall-cmd --ipset=blacklist --add-entry=192.168.1.11
firewall-cmd --ipset=blacklist --add-entry=192.168.1.12
firewall-cmd --add-rich-rule='rule source ipset=blacklist drop'
```

RHCSA: kickstart

Use the 'system-config-kickstart' utility in order to generate a Kickstart file.

If the GUI is not available, looking in /root/anaconda-ks.cfg is the next best thing as it contains the Kickstart directives that the system previously used.

Once a Kickstart method is chosen, the installer must be told where the Kickstart file is located.

ks=LOCATION is passed as an argument to the installation kernel.
ex.
```
   ks=http://server/dir/file
   ks=ftp://server/dir/file
   ks=nfs:server:/dir/file
   ks=hd:device:/dir/file
   ks=cdrom:/dir/file
```

Example Kickstart file:
```
#version=RHEL7
# System authorization information
auth --useshadow --enablemd5
# Use network installation
url --url="http://classroom.example.com/content/rhel7.0/x86_64/dvd/"
# Firewall configuration
firewall --enabled --service=ssh
firstboot --disable
ignoredisk --only-use=vda
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us','us'
# System language
lang en_US.UTF-8
# Installation logging level
logging --level=info
# Network information
network --bootproto=dhcp
# Root password
rootpw --iscrypted $6$/h/Mumvarr2dKrv1$Krv7h9.QoV0s....foMXsGXP1KllaiJ/w7EWiL1
# SELinux configuration
selinux --enforcing
# System services
services --disabled="kdump,rhsmcertd" --enabled="network,sshd,rsyslog,chronyd"
# System timezone
timezone --utc America/Los_Angeles
# System bootloader configuration
bootloader --location=mbr --boot-drive=vda
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part / --fstype="xfs" --ondisk=vda --size=10000

%packages
@core
chrony
cloud-init
dracut-config-generic
dracut-norescue
firewalld
grub2
kernel
rsync
tar
-NetworkManager
-plymouth
%end

%post --erroronfail
# For cloud images, 'eth0' _is_ the predictable device name, since
# we don't want to be tied to specific virtual (!) hardware
rm -f /etc/udev/rules.d/70*
ln -s /dev/null /etc/udev/rules.d/80-net-name-slot.rules
# simple eth0 config, again not hard-coded to the build hardware
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
USERCTL="yes"
PEERDNS="yes"
IPV6INIT="no"
EOF
%end
```
# RHCSA: libvirt

Installing libvirt:
yum install qemu-kvm qemu-img
yum install virt-manager libvirt libvirt-python python-virtinst libvirt-client

# RHCSA: yum

Create a local YUM repository:
mkdir -p /var/local && cd /var/local
cp /mnt/Packages/dcraw* .
yum -y install createrepo
createrepo /var/local

Create a definition file for the local directory:
vi /etc/yum.repos.d/local.repo
   [local]
   name=local.repo
   baseurl=file://var/local
   enabled=1
   gpgcheck=0
yum clean all
yum repolist

Adding a local repo:
```
yum-config-manager --add-repo=http://example.com
```

# RHCSA: autofs
Automatically mounting an NFS file system during runtime and system reboots (Auto File System) service.

Config file: /etc/sysconfig/autofs

Master map: /etc/auto.master which maintains entries for indirect, special and direct maps

Automounting user home directories using wildcard substitution:
* and & both are special characters to replace specific mount points
vi /etc/auto.home
   * -nfs4,rw &:/home/&
vi /etc/auto.master
   /home /etc/auto.home

Access NFS share with direct map:
yum -y install autofs
mkdir /autodir
vi /etc/auto.master
   /- /etc/auto.direct
vi /etc/auto.direct
   /autodir server1.example.com:/nfsrhcsa

Access with an indirect map:
vi /etc/auto.master
   /misc /etc/auto.misc
vi /etc/auto.misc
   autoind server1.example.com:/nfsrhcsa

To start:
   systemctl enable autofs
   systemctl start autofs
   systemctl status autofs

Automounter client config:
yum install -y autofs nfs-utils
vim /etc/auto.guests
   * -rw,nfs4 instructor.example.com:/home/guests/&
vim /etc/auto.master
   /home/guests /etc/auto.guests
systemctl enable autofs && systemctl start autofs

# RHCSA: selinux
Prevents a subject (user or process) to access an object (file, dir, fs, device, net int, port, pipe, socket, etc.) with specific access.

id -Z             # context for users
seinfo -u         # list avialble selinux users
ps -eZ            # selinux contexts for processes
ll -Z /etc/passwd # contexts for files
semanage port -l  # selinux contexts for ports

Domain transistioning: selinux allows process runing in one domain to enter another domain to eecutean application authorized to run in that domain only

Context:
chcon
matchpatchcon
restorecon
semanage

Mode:
getenforce
sestatus
setenforce

Policy:
seinfo
sesearch

Boolean:
getsebool
setsebool

Troubleshooting:
sealert

Activation / config: /etc/selinux/config

Boolean activation:
ll /sys/fs/selinux/booleans
getsebool abrt_anon_write
setsebool abrt_anon_write 1
setsebool -P abrt_anon_write on # set to persist even after reboot

Changing the SELinux context of a file:
mkdir /virtual
ls -Zd /virtual
chcon -t httpd_sys_content_t /virtual
restorecon -v /virtual # restores chcon

Using semanage to add context for a new directory:
ls -Z /virtual/
semanage fcontext -a -t httpd_sys_content_t ‘/virtual(/.*)?’
restorecon -Rfvv /virtual

Install httpd and set it to a custom location:
yum install -y httpd
mkdir /custom
vi /etc/httpd/conf/httpd.conf # set to /custom directory
systemctl start httpd
semanage fcontext -a -t httpd_sys_content_t ‘/custom(/.*)?’
restorecon -Rv /custom

Add non-standard port to selinux policy:
semanage port -l | grep htp_port # shows all prots for semanage
semanage port -a http_port_t -p tcp 8010 # adds port 8010 with type http_port_t and protocol to the policy
semanage port -d -t http_port_t -p tcp 8010 # deletes it

Copy files with and without selinux context:
cp /root/file1 /etc # without
cp --preserve=context /root/file1 /etc # with

View and analyzing selinux alerts:
tail /var/log/audit/audit.log
sealert -l UUID-HERE

Setting an enforcing mode:
getenforce
setenforce 0

Troubleshooting selinux:
yum install -y setroubleshoot-server
sealert -a /var/log/audit/audit.log # To view everything
grep 1415714880.156:29 /var/log/audit/audit.log | audit2why

Add file-context for everything under /web:
man 8 semanage-fcontext # this is found here!
semanage fcontext -a -t httpd_sys_content_t “/web(/.*)?”
restorecon -R -v /web

Modify SELinux context fo users:
useradd -Z staff_u user5 # Add user to selinux user staff_u
semanage login -a -s user_u user4 # Map existing user to SELinux user user_u
semanage login -m -S targeted -s staff_u -r s0 __default__ # all new users are added to staff_u

GUI:
yum install -y system-config-selinux

Setting an FTP boolean:
```sh
getsebool -a | grep ftp
...
ftp_home_dir --> on
...
setsebool -P ftp_home_dir on

Setting same permissions as another folder:
semanage fcontext -a -t user_home_dir_t "/xfs(/.*)?"
```

# RHCSA: console

Access virtual machine via console:
```
virt-manager
vim /etc/default/grub
   GRUB_CMDLINE_LINUX="... console=ttyS0”
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
virsh console vm.example.com
```

# RHCSA: chrony

Alternative to NTP. Very similar though.

Installing:
yum install -y chrony
systemctl enable chronyd && systemctl start chronyd

# RHCSA: ntp

Setup time first:
timedatectl
timedatectl list-timezones
timedatectl set-timezone America/Los_Angeles

Installing ntp:
yum install -y ntp
systemctl enable ntpd
systemctl start ntpd

# RCHSA: tcp_wrappers
Two files - hosts.allow and hosts.deny located in /etc are used for tcp wrapper functionality

Keywords: ALL, LOCAL, KNOWN, UNKNOWN, EXCEPT

Examples:
ALL:ALL
ALL:user1
ALL:.example.com
ALL:192.168.0.
sshd:ALL
sshd:LOCAL
vsftpd:192.168.0
vsftpd:192.168.0.0/24
vsftpd:192.168.0. EXCEPT 192.168.0.25
vsftpd:192.168.0. EXCEPT 192.168.0.25,192.168.0.26
vsftpd,sshd:user1@192.168.0.
vsftpd,sshd:user1@192.168.0.110:192.168.1.
ALL EXCEPT sshd:192.68.0.
# RCHSA: useradd
[options] username

Creating a user with no login access:
useradd -s /sbin/nologin user4

Create a user with uid 1010, home /home/user shell bash and membership in group 1001:
useradd -u 1010 -g 1001 -m -d /home/user -k /etc/skel -s /bin/bash user
/etc/skel copies basic docs to the user’s dir (skeleton)

Adding a user to a group:
useradd -G group_name username

Changing default directory:
vim /etc/default/useradd
   HOME=/usr

# RHCSA: find
find . -name newfile -print
find /dev —iname vg00*
find ~ -size -1M
