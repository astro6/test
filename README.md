

u0pengbld09p	- RHEL6
u0pengbld10p	- RHEL7

.bash_profile
sudo passsword prompt
git add --all 

git init
cronjob
puppet gits both rhel6 and 7 
staff connectivity, git 
~/test folder copy 

/data - copy
/data/reports git pull

run sample cron jobs like dns, puppet, ic-snapshot , vlan ...



rhel7:
------

1. added /data fstab entry on rhel7 disk and commented the entry.
===

2:
===
[root@u0pengbld10p:~]# mdadm --manage /dev/md0 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
[root@u0pengbld10p:~]# mdadm --manage /dev/md2 --fail /dev/sdb2
mdadm: set /dev/sdb2 faulty in /dev/md2
[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdb1[1](F) sda1[0]
      408576 blocks super 1.2 [2/1] [U_]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md2 : active raid1 sdb2[1](F) sda2[0]
      292392960 blocks super 1.2 [2/1] [U_]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# mdadm --manage /dev/md2 --remove /dev/sdb2
mdadm: hot removed /dev/sdb2 from /dev/md2
[root@u0pengbld10p:~]#  mdadm --manage /dev/md0 --remove /dev/sdb1
mdadm: hot removed /dev/sdb1 from /dev/md0
[root@u0pengbld10p:~]#


3:
===
rhel6:
------
[root@u0pengbld09p:~]# mdadm --manage /dev/md0 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0
[root@u0pengbld09p:~]# mdadm --manage /dev/md2 --fail /dev/sdb2
mdadm: set /dev/sdb2 faulty in /dev/md2
[root@u0pengbld09p:~]# mdadm --manage /dev/md2 --remove /dev/sdb2
mdadm: hot removed /dev/sdb2 from /dev/md2
[root@u0pengbld09p:~]# mdadm --manage /dev/md0 --remove /dev/sdb1
mdadm: hot removed /dev/sdb1 from /dev/md0
[root@u0pengbld09p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sda1[0]
      409536 blocks super 1.0 [2/1] [U_]

md2 : active raid1 sda2[0]
      292393984 blocks super 1.1 [2/1] [U_]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld09p:~]#


4:
===
insert rhel7 sdb disk into rhel6 slot# 2


[root@u0pengbld09p:~]# hpssacli ctrl all show config

Smart Array P440ar in Slot 0 (Embedded)   (sn: PDNLH0BRH8H2ST)


   Port Name: 1I

   Port Name: 2I

   Internal Drive Cage at Port 1I, Box 1, OK

   Internal Drive Cage at Port 2I, Box 0, OK
   array A (SAS, Unused Space: 0  MB)


      logicaldrive 1 (279.4 GB, RAID 0, OK)

      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS, 300 GB, OK)

   array B (SAS, Unused Space: 0  MB)


      logicaldrive 2 (279.4 GB, RAID 0, Failed)

      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS, 300 GB, OK)

   array C (SAS, Unused Space: 0  MB)


      logicaldrive 3 (558.9 GB, RAID 1, OK)

      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS, 600 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS, 600 GB, OK)

[root@u0pengbld09p:~]#


5.
===


[root@u0pengbld09p:~]# hpssacli ctrl slot=0 ld 2 modify reenable

Warning: Any previously existing data on the logical drive may not be valid or
         recoverable. Continue? (y/n) y

[root@u0pengbld09p:~]#

[root@u0pengbld09p:~]# hpssacli ctrl all show config

Smart Array P440ar in Slot 0 (Embedded)   (sn: PDNLH0BRH8H2ST)


   Port Name: 1I

   Port Name: 2I

   Internal Drive Cage at Port 1I, Box 1, OK

   Internal Drive Cage at Port 2I, Box 0, OK
   array A (SAS, Unused Space: 0  MB)


      logicaldrive 1 (279.4 GB, RAID 0, OK)

      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS, 300 GB, OK)

   array B (SAS, Unused Space: 0  MB)


      logicaldrive 2 (279.4 GB, RAID 0, OK)

      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS, 300 GB, OK)

   array C (SAS, Unused Space: 0  MB)


      logicaldrive 3 (558.9 GB, RAID 1, OK)

      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS, 600 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS, 600 GB, OK)

[root@u0pengbld09p:~]#


6.
===

Boot order status before reboot and Rebootthe host

[root@u0pengbld09p:~]# hpssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 1 (600508B1001C71F18560A8799B0684AB)
   Secondary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
[root@u0pengbld09p:~]#


7.  After reboot
===

[root@u0pengbld09p:~]# lsb_release -r
Release:        6.8
[root@u0pengbld09p:~]# cat /proc/mdstat
Personalities : [raid1]
md126 : active (auto-read-only) raid1 sdb2[1]
      292392960 blocks super 1.2 [2/1] [_U]
      bitmap: 0/3 pages [0KB], 65536KB chunk

md127 : active (auto-read-only) raid1 sdb1[1]
      408576 blocks super 1.2 [2/1] [_U]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md0 : active raid1 sda1[0]
      409536 blocks super 1.0 [2/1] [U_]

md2 : active raid1 sda2[0]
      292393984 blocks super 1.1 [2/1] [U_]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld09p:~]# df -kh
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg0-root   39G  1.2G   36G   4% /
tmpfs                  32G     0   32G   0% /dev/shm
/dev/md0              388M   42M  326M  12% /boot
/dev/mapper/vg0-home   48G   52M   46G   1% /home
/dev/mapper/vg0-opt    96G  524M   91G   1% /opt
/dev/mapper/vg0-tmp    16G   40M   15G   1% /tmp
/dev/mapper/vg0-var    29G  2.9G   25G  11% /var
/dev/mapper/datadg-data
                      550G  347M  522G   1% /data
[root@u0pengbld09p:~]#



8:  make rhel7/ld2 as primary boot drive and reboot the host
===

[root@u0pengbld09p:~]# hpssacli controller slot=0 ld 2 modify bootvolume=primary
[root@u0pengbld09p:~]# hpssacli controller slot=0 ld 1 modify bootvolume=secondary
[root@u0pengbld09p:~]# hpssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
   Secondary Boot Volume: logicaldrive 1 (600508B1001C71F18560A8799B0684AB)
[root@u0pengbld09p:~]#


9: after reboot , uncomment /data in fstab and reboot
===
[root@u0pengbld10p:~]# hostname
u0pengbld10p
[root@u0pengbld10p:~]# lsb_release -r
Release:        7.5
[root@u0pengbld10p:~]# df -kh
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg1-root   39G  1.5G   35G   4% /
devtmpfs               32G     0   32G   0% /dev
tmpfs                  32G     0   32G   0% /dev/shm
tmpfs                  32G  9.6M   32G   1% /run
tmpfs                  32G     0   32G   0% /sys/fs/cgroup
/dev/md0              379M  116M  243M  33% /boot
/dev/mapper/vg1-opt    96G  571M   91G   1% /opt
/dev/mapper/vg1-var    29G  3.1G   25G  12% /var
/dev/mapper/vg1-home   48G   53M   46G   1% /home
/dev/mapper/vg1-tmp    16G   41M   15G   1% /tmp
tmpfs                 6.3G     0  6.3G   0% /run/user/0
[root@u0pengbld10p:~]#
[root@u0pengbld10p:~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu May  3 20:16:31 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/vg1-root    /                       ext4    defaults        1 1
UUID=be27a249-9955-4aea-8f7a-88266c64547b /boot                   ext3    defaults        1 2
/dev/mapper/vg1-home    /home                   ext4    defaults,nodev        1 2
/dev/mapper/vg1-opt     /opt                    ext4    defaults        1 2
/dev/mapper/vg1-tmp     /tmp                    ext4    defaults        1 2
/dev/mapper/vg1-var     /var                    ext4    defaults,nodev        1 2
/dev/mapper/vg1-swap    swap                    swap    defaults        0 0
#
#
#/dev/mapper/datadg-data /data                   ext4    defaults,nodev  1 2
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md126 : active (auto-read-only) raid1 sdb2[0]
      292393984 blocks super 1.1 [2/1] [U_]
      bitmap: 2/3 pages [8KB], 65536KB chunk

md127 : active (auto-read-only) raid1 sdb1[0]
      409536 blocks super 1.0 [2/1] [U_]

md2 : active raid1 sda2[1]
      292392960 blocks super 1.2 [2/1] [_U]
      bitmap: 2/3 pages [8KB], 65536KB chunk

md0 : active raid1 sda1[1]
      408576 blocks super 1.2 [2/1] [_U]
      bitmap: 1/1 pages [4KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# pvs
  PV         VG     Fmt  Attr PSize    PFree
  /dev/md126 vg0    lvm2 a--   278.84g <32.72g
  /dev/md2   vg1    lvm2 a--   278.84g  32.75g
  /dev/sdc1  datadg lvm2 a--  <558.88g      0
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# mkdir /data

[root@u0pengbld10p:~]# mount -a
[root@u0pengbld10p:~]# df -kh
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/vg1-root      39G  1.5G   35G   4% /
devtmpfs                  32G     0   32G   0% /dev
tmpfs                     32G     0   32G   0% /dev/shm
tmpfs                     32G  9.6M   32G   1% /run
tmpfs                     32G     0   32G   0% /sys/fs/cgroup
/dev/md0                 379M  116M  243M  33% /boot
/dev/mapper/vg1-opt       96G  571M   91G   1% /opt
/dev/mapper/vg1-var       29G  3.1G   25G  12% /var
/dev/mapper/vg1-home      48G   53M   46G   1% /home
/dev/mapper/vg1-tmp       16G   41M   15G   1% /tmp
tmpfs                    6.3G     0  6.3G   0% /run/user/0
/dev/mapper/datadg-data  550G  347M  522G   1% /data
[root@u0pengbld10p:~]#


10:   after reboot 
===
[root@u0pengbld10p:~]# lsb_release -r
Release:        7.5
[root@u0pengbld10p:~]# df -kh
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/vg1-root      39G  1.5G   35G   4% /
devtmpfs                  32G     0   32G   0% /dev
tmpfs                     32G     0   32G   0% /dev/shm
tmpfs                     32G  9.7M   32G   1% /run
tmpfs                     32G     0   32G   0% /sys/fs/cgroup
/dev/md0                 379M  116M  243M  33% /boot
/dev/mapper/vg1-opt       96G  571M   91G   1% /opt
/dev/mapper/vg1-var       29G  3.1G   25G  12% /var
/dev/mapper/vg1-home      48G   53M   46G   1% /home
/dev/mapper/vg1-tmp       16G   41M   15G   1% /tmp
tmpfs                    6.3G     0  6.3G   0% /run/user/0
/dev/mapper/datadg-data  550G  347M  522G   1% /data
[root@u0pengbld10p:~]#


11:	make RHEL6/ld1 as primary boot drive and reboot 
===

[root@u0pengbld09p:~]# lsb_release -r
Release:        6.8
[root@u0pengbld09p:~]# df -kh
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg0-root   39G  1.2G   36G   4% /
tmpfs                  32G     0   32G   0% /dev/shm
/dev/md0              388M   42M  326M  12% /boot
/dev/mapper/vg0-home   48G   52M   46G   1% /home
/dev/mapper/vg0-opt    96G  524M   91G   1% /opt
/dev/mapper/vg0-tmp    16G   40M   15G   1% /tmp
/dev/mapper/vg0-var    29G  2.9G   25G  11% /var
/dev/mapper/datadg-data
                      550G  347M  522G   1% /data
[root@u0pengbld09p:~]# hpssacli ctrl all show config

Smart Array P440ar in Slot 0 (Embedded)   (sn: PDNLH0BRH8H2ST)


   Port Name: 1I

   Port Name: 2I

   Internal Drive Cage at Port 1I, Box 1, OK

   Internal Drive Cage at Port 2I, Box 0, OK
   array A (SAS, Unused Space: 0  MB)


      logicaldrive 1 (279.4 GB, RAID 0, OK)

      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS, 300 GB, OK)

   array B (SAS, Unused Space: 0  MB)


      logicaldrive 2 (279.4 GB, RAID 0, OK)

      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS, 300 GB, OK)

   array C (SAS, Unused Space: 0  MB)


      logicaldrive 3 (558.9 GB, RAID 1, OK)

      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS, 600 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS, 600 GB, OK)

[root@u0pengbld09p:~]#


12:  make RHEL7/ld2 as primary boot drive and reboot for final cutover
===

[root@u0pengbld09p:~]# hpssacli controller slot=0 ld 2 modify bootvolume=primary
[root@u0pengbld09p:~]# hpssacli controller slot=0 ld 1 modify bootvolume=secondary
[root@u0pengbld09p:~]#
[root@u0pengbld09p:~]#
[root@u0pengbld09p:~]# hpssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
   Secondary Boot Volume: logicaldrive 1 (600508B1001C71F18560A8799B0684AB)
[root@u0pengbld09p:~]#


13:	after reboot 
===

[root@u0pengbld10p:/]# lsb_release -r
Release:        7.5
[root@u0pengbld10p:/]# df -kh
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/vg1-root      39G  1.5G   35G   4% /
devtmpfs                  32G     0   32G   0% /dev
tmpfs                     32G     0   32G   0% /dev/shm
tmpfs                     32G  9.6M   32G   1% /run
tmpfs                     32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/datadg-data  550G  347M  522G   1% /data
/dev/md0                 379M  116M  243M  33% /boot
/dev/mapper/vg1-home      48G   53M   46G   1% /home
/dev/mapper/vg1-var       29G  3.1G   25G  12% /var
/dev/mapper/vg1-opt       96G  571M   91G   1% /opt
/dev/mapper/vg1-tmp       16G   41M   15G   1% /tmp
tmpfs                    6.3G     0  6.3G   0% /run/user/0
[root@u0pengbld10p:/]#


-------------------------------------------------

14:		Attach rhel6/ld1 disk with rhel7 ld2  - delete ld1 and reboot 
----------------------------------------------------------------------------
===	
	
	[root@u0pengbld10p:~]# ssacli ctrl all show config

Smart Array P440ar in Slot 0 (Embedded)   (sn: PDNLH0BRH8H2ST)



   Internal Drive Cage at Port 1I, Box 1, OK



   Internal Drive Cage at Port 2I, Box 0, OK


   Port Name: 1I

   Port Name: 2I

   Array A (SAS, Unused Space: 0  MB)

      logicaldrive 1 (279.37 GB, RAID 0, OK)

      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS HDD, 300 GB, OK)


   Array B (SAS, Unused Space: 0  MB)

      logicaldrive 2 (279.37 GB, RAID 0, OK)

      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS HDD, 300 GB, OK)


   Array C (SAS, Unused Space: 0  MB)

      logicaldrive 3 (558.88 GB, RAID 1, OK)

      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS HDD, 600 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS HDD, 600 GB, OK)

[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md126 : active (auto-read-only) raid1 sdb2[0]
      292393984 blocks super 1.1 [2/1] [U_]
      bitmap: 2/3 pages [8KB], 65536KB chunk

md127 : active (auto-read-only) raid1 sdb1[0]
      409536 blocks super 1.0 [2/1] [U_]

md0 : active raid1 sda1[1]
      408576 blocks super 1.2 [2/1] [_U]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md2 : active raid1 sda2[1]
      292392960 blocks super 1.2 [2/1] [_U]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# ssacli ctrl slot=0 ld 1 delete

Warning: Deleting an array can cause other array letters to become renamed.
         E.g. Deleting array A from arrays A,B,C will result in two remaining
         arrays A,B ... not B,C


Warning: Deleting the specified device(s) will result in data being lost.
         Continue? (y/n) y

[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# ssacli ctrl slot=0 pd 1I:1:1 modify erase; sleep 200

Warning: The erase process will begin immediately. All drive contents will be
         lost. Continue? (y/n) y

[root@u0pengbld10p:~]# ssacli ctrl slot=0 pd 1I:1:1 modify stoperase
[root@u0pengbld10p:~]#


15:  after reboot status
===

[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sda1[1]
      408576 blocks super 1.2 [2/1] [_U]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md2 : active raid1 sda2[1]
      292392960 blocks super 1.2 [2/1] [_U]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# ssacli ctrl all show config

Smart Array P440ar in Slot 0 (Embedded)   (sn: PDNLH0BRH8H2ST)



   Internal Drive Cage at Port 1I, Box 1, OK



   Internal Drive Cage at Port 2I, Box 0, OK


   Port Name: 1I

   Port Name: 2I

   Array A (SAS, Unused Space: 0  MB)

      logicaldrive 2 (279.37 GB, RAID 0, OK)

      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS HDD, 300 GB, OK)


   Array B (SAS, Unused Space: 0  MB)

      logicaldrive 3 (558.88 GB, RAID 1, OK)

      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS HDD, 600 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS HDD, 600 GB, OK)

   Unassigned

      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS HDD, 300 GB, OK)

[root@u0pengbld10p:~]#


16:  create ld1 and attach to rhel7/ld2 disk
===

[root@u0pengbld10p:~]# ssacli ctrl slot=0 create type=ld drives=1I:1:1 raid=0

Warning: Creation of this logical drive has caused array letters to become
         renamed.

[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# fdisk -l /dev/sdc

Disk /dev/sdc: 300.0 GB, 299966445568 bytes, 585871964 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 262144 bytes / 262144 bytes

[root@u0pengbld10p:~]# sfdisk -d /dev/sda | sfdisk /dev/sdc --force
Checking that no-one is using this disk right now ...
OK

Disk /dev/sdc: 36468 cylinders, 255 heads, 63 sectors/track
sfdisk:  /dev/sdc: unrecognized partition table type

Old situation:
sfdisk: No partitions found

New situation:
Units: sectors of 512 bytes, counting from 0

   Device Boot    Start       End   #sectors  Id  System
/dev/sdc1   *      2048    821247     819200  fd  Linux raid autodetect
/dev/sdc2        821248 585871359  585050112  fd  Linux raid autodetect
/dev/sdc3             0         -          0   0  Empty
/dev/sdc4             0         -          0   0  Empty
Warning: partition 1 does not end at a cylinder boundary
Warning: partition 2 does not start at a cylinder boundary
Warning: partition 2 does not end at a cylinder boundary
Successfully wrote the new partition table

Re-reading the partition table ...

If you created or changed a DOS partition, /dev/foo7, say, then use dd(1)
to zero the first 512 bytes:  dd if=/dev/zero of=/dev/foo7 bs=512 count=1
(See fdisk(8).)
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# mdadm --manage /dev/md0 --add /dev/sdc1
mdadm: added /dev/sdc1
[root@u0pengbld10p:~]# mdadm --manage /dev/md2 --add /dev/sdc2
mdadm: added /dev/sdc2
[root@u0pengbld10p:~]#
[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdc1[2] sda1[1]
      408576 blocks super 1.2 [2/2] [UU]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md2 : active raid1 sdc2[2] sda2[1]
      292392960 blocks super 1.2 [2/1] [_U]
      [>....................]  recovery =  0.2% (688832/292392960) finish=21.1min speed=229610K/sec
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#


[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdc1[2] sda1[1]
      408576 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md2 : active raid1 sdc2[2] sda2[1]
      292392960 blocks super 1.2 [2/2] [UU]
      bitmap: 1/3 pages [4KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#



17:  install boot record on newly created boot disk
===	 make ld1 as primary boot drive
     reboot
	 

[root@u0pengbld10p:~]# grub2-install /dev/sdc
Installing for i386-pc platform.
Installation finished. No error reported.
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# ssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
   Secondary Boot Volume: None
         Boot Volume: Primary
[root@u0pengbld10p:~]#

[root@u0pengbld10p:~]# ssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 1 (600508B1001CA08B00DC045F4662AEF4)
   Secondary Boot Volume: None
         Boot Volume: Primary
[root@u0pengbld10p:~]# ssacli controller slot=0 ld 2 modify bootvolume=secondary
[root@u0pengbld10p:~]#
[root@u0pengbld10p:~]#
[root@u0pengbld10p:~]# ssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 1 (600508B1001CA08B00DC045F4662AEF4)
   Secondary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
         Boot Volume: Primary
         Boot Volume: Secondary
[root@u0pengbld10p:~]#


18:   After final reboot 
===

[root@u0pengbld10p:~]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdb1[1] sda1[2]
      408576 blocks super 1.2 [2/2] [UU]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md2 : active raid1 sdb2[1] sda2[2]
      292392960 blocks super 1.2 [2/2] [UU]
      bitmap: 2/3 pages [8KB], 65536KB chunk

unused devices: <none>
[root@u0pengbld10p:~]#
[root@u0pengbld10p:~]# ssacli ctrl all show config detail | grep 'Boot '
   Primary Boot Volume: logicaldrive 1 (600508B1001CA08B00DC045F4662AEF4)
   Secondary Boot Volume: logicaldrive 2 (600508B1001C5E570300CFFD117C8F63)
         Boot Volume: Primary
         Boot Volume: Secondary
[root@u0pengbld10p:~]#



==========================================


/usr/sbin/dmidecode -s system-manufacturer


ssacli ctrl slot=0 show config | grep 'logicaldrive 1' | grep 'RAID 0'


ssacli ctrl slot=0 ld 1 show detail


[root@u0pengbld09p:~]# ssacli ctrl slot=0 ld 1 show detail

Smart Array P440ar in Slot 0 (Embedded)

   Array A

      Logical Drive: 1
         Size: 279.37 GB
         Fault Tolerance: 0
         Heads: 255
         Sectors Per Track: 32
         Cylinders: 65535
         Strip Size: 256 KB
         Full Stripe Size: 256 KB
         Status: OK
         MultiDomain Status: OK
         Caching:  Enabled
         Unique Identifier: 600508B1001CA08B00DC045F4662AEF4
         Disk Name: /dev/sdb
         Mount Points: None
         Boot Volume: Secondary
         Logical Drive Label: 0130D4E4PDNLH0BRH8H2ST6BF2
         Drive Type: Data
         LD Acceleration Method: Controller Cache

ssacli ctrl slot=0 ld 1 modify bootvolume=none


		 
















