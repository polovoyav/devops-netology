# 03-sysadmin-05-fs  

## 1)  
Прочитал про sparse файлы. Вариант для бекапов и виртуальных дисков при ограниченных дисковых ресурсах.

## 2)  
Нет, не могут. Hardlink по сути является тем же файлом, на который ссылается, имеет тот же индексный дескриптор  
inode, который как раз и содержит информацию о файле, в том числе о владельце/группе и о правах доступа к нему.  

>vagrant@sandbox:\~/test$ touch file1  
>vagrant@sandbox:\~/test$ ls -lhi file1  
>131099 -rw-rw-r-- 1 vagrant vagrant 0 Jan 23 13:38 file1  

Создали файл с `inode 131099`.  

>vagrant@sandbox:\~/test$ ln file1 hardlink1  
>vagrant@sandbox:\~/test$ ln file1 hardlink2  

>vagrant@sandbox:\~/test$ ls -lhi | grep 131099  
>131099 -rw-rw-r-- 3 vagrant vagrant  0 Jan 23 13:38 file1  
>131099 -rw-rw-r-- 3 vagrant vagrant  0 Jan 23 13:38 hardlink1  
>131099 -rw-rw-r-- 3 vagrant vagrant  0 Jan 23 13:38 hardlink2  

Хардлинки имеют тот же inode, набор прав и владельца.  

>vagrant@sandbox:\~/test$ chmod ug+x file1  
>vagrant@sandbox:\~/test$ sudo chown root file1  
>vagrant@sandbox:\~/test$ ls -lhi | grep 131099  
>131099 -rwxrwxr-- 3 vagrant vagrant  0 Jan 23 13:38 file1  
>131099 -rwxrwxr-- 3 vagrant vagrant  0 Jan 23 13:38 hardlink1  
>131099 -rwxrwxr-- 3 vagrant vagrant  0 Jan 23 13:38 hardlink2  

Набор прав и владелец сменились на файле и хардлинках.

## 3)  
Сделано.  

>vagrant@sandbox2:~$ lsblk  
>NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT  
>sda                         8:0    0   64G  0 disk  
>├─sda1                      8:1    0    1M  0 part  
>├─sda2                      8:2    0    1G  0 part /boot  
>└─sda3                      8:3    0   63G  0 part  
>  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /  
>sdb                         8:16   0  2.5G  0 disk  
>sdc                         8:32   0  2.5G  0 disk  

Отображение loop-устройств убрал через создание алиаса `alias lsblk="lsblk -e 7"`  

## 4)  
Сделано.  
>vagrant@sandbox2:~$ sudo fdisk /dev/sdb  
>  
>Welcome to fdisk (util-linux 2.34).  
>Changes will remain in memory only, until you decide to write them.  
>Be careful before using the write command.  
>  
>Device does not contain a recognized partition table.  
>Created a new DOS disklabel with disk identifier 0x0cfc9037.  
  
...  
  
>Command (m for help): p  
>Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors  
>Disk model: VBOX HARDDISK  
>Units: sectors of 1 * 512 = 512 bytes  
>Sector size (logical/physical): 512 bytes / 512 bytes  
>I/O size (minimum/optimal): 512 bytes / 512 bytes  
>Disklabel type: dos  
>Disk identifier: 0xdaa8af1f  
>  
>Device     Boot   Start     End Sectors  Size Id Type  
>/dev/sdb1          2048 4196351 4194304    2G 83 Linux  
>/dev/sdb2       4196352 5242879 1046528  511M 83 Linux  
>  
>Command (m for help): w  
>  
>The partition table has been altered.  
>Calling ioctl() to re-read partition table.  
>Syncing disks.  
  
## 5)  
>vagrant@sandbox2:\~$ sudo -i  
>root@sandbox2:\~#  
>root@sandbox2:\~# sfdisk -d /dev/sdb | sfdisk /dev/sdc  
>Checking that no-one is using this disk right now ... OK  
>  
>Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors  
>Disk model: VBOX HARDDISK  
>Units: sectors of 1 * 512 = 512 bytes  
>Sector size (logical/physical): 512 bytes / 512 bytes  
>I/O size (minimum/optimal): 512 bytes / 512 bytes  
>Disklabel type: dos  
>Disk identifier: 0xa8b7ab1d  
>  
>Old situation:  
>  
> >>> Script header accepted.  
> >>> Script header accepted.  
> >>> Script header accepted.  
> >>> Script header accepted.  
> >>> Created a new DOS disklabel with disk identifier 0xdaa8af1f.  
>/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.  
>/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.  
>/dev/sdc3: Done.  
>  
>New situation:  
>Disklabel type: dos  
>Disk identifier: 0xdaa8af1f  
>  
>Device     Boot   Start     End Sectors  Size Id Type  
>/dev/sdc1          2048 4196351 4194304    2G 83 Linux  
>/dev/sdc2       4196352 5242879 1046528  511M 83 Linux  
>  
>The partition table has been altered.  
>Calling ioctl() to re-read partition table.  
>Syncing disks.  

Таблица разделов:  
>vagrant@sandbox2:~$ lsblk  
>NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT  
>sda                         8:0    0   64G  0 disk  
>├─sda1                      8:1    0    1M  0 part  
>├─sda2                      8:2    0    1G  0 part /boot  
>└─sda3                      8:3    0   63G  0 part  
>  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /  
>sdb                         8:16   0  2.5G  0 disk  
>├─sdb1                      8:17   0    2G  0 part  
>└─sdb2                      8:18   0  511M  0 part  
>sdc                         8:32   0  2.5G  0 disk  
>├─sdc1                      8:33   0    2G  0 part  
>└─sdc2                      8:34   0  511M  0 part  

## 6)  
>root@sandbox2:\~# mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1  
>mdadm: Note: this array has metadata at the start and  
>    may not be suitable as a boot device.  If you plan to  
>    store '/boot' on this device please ensure that  
>    your boot-loader understands md/v1.x metadata, or use  
>    --metadata=0.90  
>mdadm: size set to 2094080K  
>Continue creating array? y  
>mdadm: Defaulting to version 1.2 metadata  
>mdadm: array /dev/md0 started.  
>root@sandbox2:\~# cat /proc/mdstat  
>Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]  
>md0 : active raid1 sdc1[1] sdb1[0]  
>      2094080 blocks super 1.2 [2/2] [UU]  
>      [=================>...]  resync = 86.4% (1810176/2094080) finish=0.0min speed=201130K/sec  
>  
>unused devices: <none>  

Видим, что процесс создания raid идет.  

## 7)  
>root@sandbox2:\~# mdadm --create --verbose /dev/md1 --level=0 --raid-devices=2 /dev/  
>sdb2 /dev/sdc2  
>mdadm: chunk size defaults to 512K  
>mdadm: Defaulting to version 1.2 metadata  
>mdadm: array /dev/md1 started.  
>root@sandbox2:\~# cat /proc/mdstat  
>Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]  
>md1 : active raid0 sdc2[1] sdb2[0]  
>      1042432 blocks super 1.2 512k chunks  
>  
>md0 : active raid1 sdc1[1] sdb1[0]  
>      2094080 blocks super 1.2 [2/2] [UU]  
>  
>unused devices: <none>  

## 8)  
>root@sandbox2:\~# pvcreate /dev/md0 /dev/md1  
>  Physical volume "/dev/md0" successfully created.  
>  Physical volume "/dev/md1" successfully created.  
>  
>root@sandbox2:\~# pvdisplay  
>  --- Physical volume ---  
>  PV Name               /dev/sda3  
>  VG Name               ubuntu-vg  
>  PV Size               <63.00 GiB / not usable 0  
>  Allocatable           yes  
>  PE Size               4.00 MiB  
>  Total PE              16127  
>  Free PE               8063  
>  Allocated PE          8064  
>  PV UUID               sDUvKe-EtCc-gKuY-ZXTD-1B1d-eh9Q-XldxLf  
>  
>  "/dev/md0" is a new physical volume of "<2.00 GiB"  
>  --- NEW Physical volume ---  
>  PV Name               /dev/md0  
>  VG Name  
>  PV Size               <2.00 GiB  
>  Allocatable           NO  
>  PE Size               0  
>  Total PE              0  
>  Free PE               0  
>  Allocated PE          0  
>  PV UUID               Tzu8Of-zJH7-Hpdw-31O1-QKHx-g3Kv-iUQjxh  

## 9)  
>root@sandbox2:\~# vgcreate vg01 /dev/md0 /dev/md1  
>  Volume group "vg01" successfully created  
>  
>root@sandbox2:\~# vgdisplay vg01  
>  --- Volume group ---  
>  VG Name               vg01  
>  System ID  
>  Format                lvm2  
>  Metadata Areas        2  
>  Metadata Sequence No  1  
>  VG Access             read/write  
>  VG Status             resizable  
>  MAX LV                0  
>  Cur LV                0  
>  Open LV               0  
>  Max PV                0  
>  Cur PV                2  
>  Act PV                2  
>  VG Size               <2.99 GiB  
>  PE Size               4.00 MiB  
>  Total PE              765  
>  Alloc PE / Size       0 / 0  
>  Free  PE / Size       765 / <2.99 GiB  
>  VG UUID               UMgjdQ-57ZD-QT2Q-IF5i-SKc9-PHrJ-ufxeLx  

## 10)  
Примечание для себя: To control which PVs a new LV will use, specify one or more PVs as position args at  
the end of the command line. lvcreate will allocate physical extents only from the specified PVs.  

>root@sandbox2:\~# lvcreate -L 100 -n lv01 vg01 /dev/md1  
>  Logical volume "lv01" created.  
>  
>root@sandbox2:\~# lvdisplay  /dev/vg01/lv01  
>  --- Logical volume ---  
>  LV Path                /dev/vg01/lv01  
>  LV Name                lv01  
>  VG Name                vg01  
>  LV UUID                BsJpkH-dEH6-KzZR-JCS1-40Qt-n3S0-IoJ4Oz  
>  LV Write Access        read/write  
>  LV Creation host, time sandbox2, 2022-01-24 16:20:42 +0000  
>  LV Status              available  
>  # open                 0  
>  LV Size                100.00 MiB  
>  Current LE             25  
>  Segments               1  
>  Allocation             inherit  
>  Read ahead sectors     auto  
>  - currently set to     4096  
>  Block device           253:1  

Как итог:  
>root@sandbox2:\~# pvs  
>  PV         VG        Fmt  Attr PSize    PFree  
>  /dev/md0   vg01      lvm2 a--    <2.00g  <2.00g  
>  /dev/md1   vg01      lvm2 a--  1016.00m 916.00m  
>  /dev/sda3  ubuntu-vg lvm2 a--   <63.00g <31.50g  
>root@sandbox2:\~# vgs  
>  VG        #PV #LV #SN Attr   VSize   VFree  
>  ubuntu-vg   1   1   0 wz--n- <63.00g <31.50g  
>  vg01        2   1   0 wz--n-  <2.99g   2.89g  
>root@sandbox2:\~# lvs  
>  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
>  ubuntu-lv ubuntu-vg -wi-ao----  31.50g  
>  lv01      vg01      -wi-a----- 100.00m  

## 11)  
>root@sandbox2:~# mkfs.ext4 /dev/vg01/lv01  
>mke2fs 1.45.5 (07-Jan-2020)  
>Creating filesystem with 25600 4k blocks and 25600 inodes  
>  
>Allocating group tables: done  
>Writing inode tables: done  
>Creating journal (1024 blocks): done  
>Writing superblocks and filesystem accounting information: done  

## 12)  
>root@sandbox2:\~# mkdir /tmp/new  
>root@sandbox2:\~# mount /dev/vg01/lv01 /tmp/new  
>root@sandbox2:\~# lsblk --fs  
>  
>NAME FSTYPE LABEL  UUID                                   FSAVAIL FSUSE% MOUNTPOINT  
...  

>sdb  
...  
>└─sdb2  
>     linux_ sandbox2:1  
>                   bc42faea-6d9f-fb9f-a739-7750e9031876  
>  └─md1  
>     LVM2_m        mOcxgB-MZBo-PWBe-gwf4-nzX1-e9Uu-DmBd96  
>    └─vg01-lv01  
>       ext4          8ea88ba4-40c3-4fc2-a1bd-f5de769be660     85.8M     0% /tmp/new  
>sdc  
...  
>└─sdc2  
>     linux_ sandbox2:1  
>                   bc42faea-6d9f-fb9f-a739-7750e9031876  
>  └─md1  
>     LVM2_m        mOcxgB-MZBo-PWBe-gwf4-nzX1-e9Uu-DmBd96  
>    └─vg01-lv01  
>       ext4          8ea88ba4-40c3-4fc2-a1bd-f5de769be660     85.8M     0% /tmp/new  

## 13)  
>root@sandbox2:~# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz  
>--2022-01-24 17:25:09--  https://mirror.yandex.ru/ubuntu/ls-lR.gz  
>Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183  
>Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.  
>HTTP request sent, awaiting response... 200 OK  
>Length: 21989447 (21M) [application/octet-stream]  
>Saving to: ‘/tmp/new/test.gz’  
>  
>/tmp/new/test.gz     100%[=====================>]  20.97M   747KB/s    in 47s  
>  
>2022-01-24 17:25:56 (459 KB/s) - ‘/tmp/new/test.gz’ saved [21989447/21989447]  

## 14)  
>root@sandbox2:~# lsblk  
>NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT  
>sda                         8:0    0   64G  0 disk  
>├─sda1                      8:1    0    1M  0 part  
>├─sda2                      8:2    0    1G  0 part  /boot  
>└─sda3                      8:3    0   63G  0 part  
>  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /  
>sdb                         8:16   0  2.5G  0 disk  
>├─sdb1                      8:17   0    2G  0 part  
>│ └─md0                     9:0    0    2G  0 raid1  
>└─sdb2                      8:18   0  511M  0 part  
>  └─md1                     9:1    0 1018M  0 raid0  
>    └─vg01-lv01           253:1    0  100M  0 lvm   /tmp/new  
>sdc                         8:32   0  2.5G  0 disk  
>├─sdc1                      8:33   0    2G  0 part  
>│ └─md0                     9:0    0    2G  0 raid1  
>└─sdc2                      8:34   0  511M  0 part  
>  └─md1                     9:1    0 1018M  0 raid0  
>    └─vg01-lv01           253:1    0  100M  0 lvm   /tmp/new  

## 15)  
Примечание для себя: После исполнения команды, переменная $? хранит код завершения последней команды.  
>root@sandbox2:\~# gzip -t /tmp/new/test.gz  
>root@sandbox2:~# echo $?  
>0  

## 16)  
Примечание: Use a specific destination PV when moving physical extents: pvmove /dev/sdb1 /dev/sdc1  
  
>root@sandbox2:\~# pvmove /dev/md1 /dev/md0  
>  /dev/md1: Moved: 52.00%  
>  /dev/md1: Moved: 100.00%  
>root@sandbox2:\~# lsblk  
>NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT  
>sda                         8:0    0   64G  0 disk  
>├─sda1                      8:1    0    1M  0 part  
>├─sda2                      8:2    0    1G  0 part  /boot  
>└─sda3                      8:3    0   63G  0 part  
>  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /  
>sdb                         8:16   0  2.5G  0 disk  
>├─sdb1                      8:17   0    2G  0 part  
>│ └─md0                     9:0    0    2G  0 raid1  
>│   └─vg01-lv01           253:1    0  100M  0 lvm   /tmp/new  
>└─sdb2                      8:18   0  511M  0 part  
>  └─md1                     9:1    0 1018M  0 raid0  
>sdc                         8:32   0  2.5G  0 disk  
>├─sdc1                      8:33   0    2G  0 part  
>│ └─md0                     9:0    0    2G  0 raid1  
>│   └─vg01-lv01           253:1    0  100M  0 lvm   /tmp/new  
>└─sdc2                      8:34   0  511M  0 part  
>  └─md1                     9:1    0 1018M  0 raid0  

## 17)  
>root@sandbox2:~# mdadm --detail /dev/md0  
>/dev/md0:  
>           Version : 1.2  
>     Creation Time : Sun Jan 23 20:59:56 2022  
>        Raid Level : raid1  
>        Array Size : 2094080 (2045.00 MiB 2144.34 MB)  
>     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)  
>      Raid Devices : 2  
>     Total Devices : 2  
>       Persistence : Superblock is persistent  
>  
>       Update Time : Mon Jan 24 18:10:59 2022  
>             State : clean  
>    Active Devices : 2  
>   Working Devices : 2  
>    Failed Devices : 0  
>     Spare Devices : 0  
>  
>Consistency Policy : resync  
>  
>              Name : sandbox2:0  (local to host sandbox2)  
>              UUID : 56ab26ea:63949c5e:31c3954f:0ebd1177  
>            Events : 17  
>  
>    Number   Major   Minor   RaidDevice State  
>       0       8       17        0      active sync   /dev/sdb1  
>       1       8       33        1      active sync   /dev/sdc1  
	   
>root@sandbox2:~# mdadm /dev/md0 --fail /dev/sdb1  
>mdadm: set /dev/sdb1 faulty in /dev/md0  

>root@sandbox2:~# mdadm --detail /dev/md0  
>/dev/md0:  
>           Version : 1.2  
>     Creation Time : Sun Jan 23 20:59:56 2022  
>        Raid Level : raid1  
>        Array Size : 2094080 (2045.00 MiB 2144.34 MB)  
>     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)  
>      Raid Devices : 2  
>     Total Devices : 2  
>       Persistence : Superblock is persistent  
>  
>       Update Time : Mon Jan 24 18:23:38 2022  
>             State : clean, degraded  
>    Active Devices : 1  
>   Working Devices : 1  
>    Failed Devices : 1  
>     Spare Devices : 0  
>  
>Consistency Policy : resync  
>  
>              Name : sandbox2:0  (local to host sandbox2)  
>              UUID : 56ab26ea:63949c5e:31c3954f:0ebd1177  
>            Events : 19  
>  
>    Number   Major   Minor   RaidDevice State  
>       -       0        0        0      removed  
>       1       8       33        1      active sync   /dev/sdc1  
>  
>       0       8       17        -      faulty   /dev/sdb1  

## 18)  
>root@sandbox2:~# dmesg -H | grep md0  
>[Jan24 14:29] md/raid1:md0: not clean -- starting background reconstruction  
>[  +0.000002] md/raid1:md0: active with 2 out of 2 mirrors  
>[  +0.000017] md0: detected capacity change from 0 to 2144337920  
>[  +0.004825] md: resync of RAID array md0  
>[ +10.435490] md: md0: resync done.  
>[Jan24 18:23] md/raid1:md0: Disk failure on sdb1, disabling device.  
>              md/raid1:md0: Operation continuing on 1 devices.  

## 19)  
>root@sandbox2:\~# gzip -t /tmp/new/test.gz  
>root@sandbox2:\~# echo $?  
>0  

## 20)  
>PS C:\Users\andrew\VagrantVM> vagrant destroy sandbox2  
>    sandbox2: Are you sure you want to destroy the 'sandbox2' VM? [y/N] y  
> ==> sandbox2: Forcing shutdown of VM...  
> ==> sandbox2: Destroying VM and associated drives...  




