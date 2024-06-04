### lsblk 查看磁盘
```
lsblk 查看磁盘，可以扩容
[root@CP-C11-T mysql]# pvcreate /dev/sdb    /c创建
  Physical volume "/dev/sdb" successfully created.
[root@CP-C11-T mysql]# pvs
  PV         VG     Fmt  Attr PSize   PFree  
  /dev/sda2  rootvg lvm2 a--  <63.00g      0 
  /dev/sdb          lvm2 ---  200.00g 200.00g
[root@CP-C11-T mysql]# vgcreate rootvg /dev/sdb  新增vg，rootvg这个名字可变的。
[root@CP-C11-T mysql]# vgextend rootvg /dev/sdb  扩容
  Volume group "rootvg" successfully extended
[root@CP-C11-T mysql]# 
[root@CP-C11-T mysql]# 
[root@CP-C11-T mysql]# vgs  查看多长
  VG     #PV #LV #SN Attr   VSize   VFree   
  rootvg   2   5   0 wz--n- 262.99g <200.00g
[root@CP-C11-T mysql]# lvextend -L 100G /dev/rootvg/lvvar 全部扩容给某块磁盘
[root@CP-C11-T mysql]# lvextend -L 100G /dev/rootvg/lvvar  扩容，lvextend -L +100G /dev/rootvg/lvvar #加100G
  Size of logical volume rootvg/lvvar changed from 2.00 GiB (512 extents) to 100.00 GiB (25600 extents).
  Logical volume rootvg/lvvar successfully resized.
[root@CP-C11-T mysql]# 
[root@CP-C11-T mysql]# xfs_growfs /dev/rootvg/lvvar 
meta-data=/dev/mapper/rootvg-lvvar isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 26214400
[root@CP-C11-T mysql]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-lvroot   30G  6.3G   24G  21% /
devtmpfs                   7.9G     0  7.9G   0% /dev
tmpfs                      7.9G     0  7.9G   0% /dev/shm
tmpfs                      7.9G   41M  7.8G   1% /run
tmpfs                      7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/sda1                 1014M  160M  855M  16% /boot
/dev/mapper/rootvg-lvvar   100G  2.0G   98G   2% /var
/dev/mapper/rootvg-lvopt    10G  556M  9.5G   6% /opt
/dev/mapper/rootvg-lvhome 1014M   54M  961M   6% /home
tmpfs                      1.6G   12K  1.6G   1% /run/user/42
tmpfs  
```

### 验证

```
[sysadm@C112006222 ~]$ su - root
Password: 
Last login: Fri Jun  5 16:49:19 CST 2020 on pts/0
[root@C112006222 ~]# lsblk   ##查看哪些可以扩容
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                 8:0    0   64G  0 disk 
├─sda1              8:1    0    1G  0 part /boot
└─sda2              8:2    0   63G  0 part 
  ├─rootvg-lvroot 253:0    0   30G  0 lvm  /
  ├─rootvg-lvswap 253:1    0   20G  0 lvm  [SWAP]
  ├─rootvg-lvvar  253:2    0    2G  0 lvm  /var
  ├─rootvg-lvhome 253:3    0    1G  0 lvm  /home
  └─rootvg-lvopt  253:4    0   10G  0 lvm  /opt
sdb                 8:16   0  500G  0 disk 
sr0                11:0    1 1024M  0 rom  
[root@C112006222 ~]# pvcreate /dev/sdb  ## 加入 pvcreate模式
  Physical volume "/dev/sdb" successfully created.
[root@C112006222 ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree  
  /dev/sda2  rootvg lvm2 a--  <63.00g      0 
  /dev/sdb          lvm2 ---  500.00g 500.00g

[root@C112006222 ~]# vgextend  rootvg /dev/sdb   ## 把/dev/sdb加到rootvg
  Volume group "rootvg" successfully extended
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree   
  rootvg   2   5   0 wz--n- 562.99g <500.00g
[root@C112006222 ~]# lvextend -L 50G /dev/rootvg/lvvar ## 把var目录扩容到50G
  Size of logical volume rootvg/lvvar changed from 2.00 GiB (512 extents) to 50.00 GiB (12800 extents).
  Logical volume rootvg/lvvar successfully resized.
[root@C112006222 ~]# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                 8:0    0   64G  0 disk 
├─sda1              8:1    0    1G  0 part /boot
└─sda2              8:2    0   63G  0 part 
  ├─rootvg-lvroot 253:0    0   30G  0 lvm  /
  ├─rootvg-lvswap 253:1    0   20G  0 lvm  [SWAP]
  ├─rootvg-lvvar  253:2    0   50G  0 lvm  /var
  ├─rootvg-lvhome 253:3    0    1G  0 lvm  /home
  └─rootvg-lvopt  253:4    0   10G  0 lvm  /opt
sdb                 8:16   0  500G  0 disk 
└─rootvg-lvvar    253:2    0   50G  0 lvm  /var
sr0                11:0    1 1024M  0 rom  
[root@C112006222 ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-lvroot   30G  4.4G   26G  15% /
devtmpfs                    16G     0   16G   0% /dev
tmpfs                       16G     0   16G   0% /dev/shm
tmpfs                       16G  8.9M   16G   1% /run
tmpfs                       16G     0   16G   0% /sys/fs/cgroup
/dev/sda1                 1014M  160M  855M  16% /boot
/dev/mapper/rootvg-lvvar   2.0G  254M  1.8G  13% /var
/dev/mapper/rootvg-lvopt    10G   33M   10G   1% /opt
/dev/mapper/rootvg-lvhome 1014M   37M  978M   4% /home
tmpfs                      3.2G   12K  3.2G   1% /run/user/42
tmpfs                      3.2G     0  3.2G   0% /run/user/1000
[root@C112006222 ~]# xfs_growfs /dev/rootvg/lvvar   ## 生效下var
meta-data=/dev/mapper/rootvg-lvvar isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 13107200
[root@C112006222 ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-lvroot   30G  4.4G   26G  15% /
devtmpfs                    16G     0   16G   0% /dev
tmpfs                       16G     0   16G   0% /dev/shm
tmpfs                       16G  8.9M   16G   1% /run
tmpfs                       16G     0   16G   0% /sys/fs/cgroup
/dev/sda1                 1014M  160M  855M  16% /boot
/dev/mapper/rootvg-lvvar    50G  257M   50G   1% /var
/dev/mapper/rootvg-lvopt    10G   33M   10G   1% /opt
/dev/mapper/rootvg-lvhome 1014M   37M  978M   4% /home
tmpfs                      3.2G   12K  3.2G   1% /run/user/42
tmpfs                      3.2G     0  3.2G   0% /run/user/1000
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# lvcreate -L 300G -n lvdata rootvg  ## 创建300G的/data
  Logical volume "lvdata" created.
[root@C112006222 ~]# lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lvdata rootvg -wi-a----- 300.00g                                                    
  lvhome rootvg -wi-ao----   1.00g                                                    
  lvopt  rootvg -wi-ao----  10.00g                                                    
  lvroot rootvg -wi-ao---- <30.00g                                                    
  lvswap rootvg -wi-ao----  20.00g                                                    
  lvvar  rootvg -wi-ao----  50.00g                                                    
[root@C112006222 ~]# mkfs.xfs /dev/rootvg/lvdata   ## 格式化data
meta-data=/dev/rootvg/lvdata     isize=512    agcount=4, agsize=19660800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=78643200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=38400, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# 
[root@C112006222 ~]# ls /
bin  boot  changepw.sh  dev  etc  home  lib  lib64  lock_root_sshlogin.sh  media  mklv.sh  mkvg.sh  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@C112006222 ~]# mkdir /data
[root@C112006222 ~]# vi /etc/fstab 
/dev/mapper/rootvg-lvdata /data xfs defaults  0 0
[root@C112006222 ~]# mount -a
[root@C112006222 ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-lvroot   30G  4.4G   26G  15% /
devtmpfs                    16G     0   16G   0% /dev
tmpfs                       16G     0   16G   0% /dev/shm
tmpfs                       16G  8.9M   16G   1% /run
tmpfs                       16G     0   16G   0% /sys/fs/cgroup
/dev/sda1                 1014M  160M  855M  16% /boot
/dev/mapper/rootvg-lvvar    50G  257M   50G   1% /var
/dev/mapper/rootvg-lvopt    10G   33M   10G   1% /opt
/dev/mapper/rootvg-lvhome 1014M   37M  978M   4% /home
tmpfs                      3.2G   12K  3.2G   1% /run/user/42
tmpfs                      3.2G     0  3.2G   0% /run/user/1000
/dev/mapper/rootvg-lvdata  300G   33M  300G   1% /data
[root@C112006222 ~]# df -hi
Filesystem                Inodes IUsed IFree IUse% Mounted on
/dev/mapper/rootvg-lvroot    15M  163K   15M    2% /
devtmpfs                    4.0M   414  4.0M    1% /dev
tmpfs                       4.0M     1  4.0M    1% /dev/shm
tmpfs                       4.0M   569  4.0M    1% /run
tmpfs                       4.0M    16  4.0M    1% /sys/fs/cgroup
/dev/sda1                   512K   327  512K    1% /boot
/dev/mapper/rootvg-lvvar     25M  9.9K   25M    1% /var
/dev/mapper/rootvg-lvopt    5.0M     4  5.0M    1% /opt
/dev/mapper/rootvg-lvhome   512K   195  512K    1% /home
tmpfs                       4.0M     9  4.0M    1% /run/user/42
tmpfs                       4.0M     1  4.0M    1% /run/user/1000
/dev/mapper/rootvg-lvdata   150M     3  150M    1% /data


-----------------------
### 剩余的
mkfs.ext4 -T largefile /dev/sdb2 &

```



### 实际操作

```
lsblk

[root@JCGY-postgres-2 ~]# lsblk
NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                 8:0    0   64G  0 disk
├─sda1              8:1    0    1G  0 part /boot
└─sda2              8:2    0   63G  0 part
  ├─rootvg-lvroot 253:0    0   30G  0 lvm  /
  ├─rootvg-lvswap 253:1    0   20G  0 lvm  [SWAP]
  ├─rootvg-lvvar  253:2    0    2G  0 lvm  /var
  ├─rootvg-lvhome 253:3    0    1G  0 lvm  /home
  └─rootvg-lvopt  253:4    0   10G  0 lvm  /opt
sdb                 8:16   0  200G  0 disk
sr0                11:0    1 1024M  0 rom

pvcreate /dev/sdb  ## 加入 pvcreate模式


[root@JCGY-postgres-2 ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
  
[root@JCGY-postgres-2 ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree
  /dev/sda2  rootvg lvm2 a--  <63.00g      0
  /dev/sdb          lvm2 ---  200.00g 200.00g


vgextend  rootvg /dev/sdb
[root@JCGY-postgres-2 ~]# vgextend  rootvg /dev/sdb
  Volume group "rootvg" successfully extended

[root@JCGY-postgres-2 ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  rootvg   2   5   0 wz--n- 262.99g <200.00g


lvcreate -L 190G -n lvdata rootvg  ## 创建300G的/data

[root@JCGY-postgres-2 ~]# lvcreate -L 190G -n lvdata rootvg
  Logical volume "lvdata" created.



[root@JCGY-postgres-2 ~]# lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lvdata rootvg -wi-a----- 190.00g
  lvhome rootvg -wi-ao----   1.00g
  lvopt  rootvg -wi-ao----  10.00g
  lvroot rootvg -wi-ao---- <30.00g
  lvswap rootvg -wi-ao----  20.00g
  lvvar  rootvg -wi-ao----   2.00g

mkfs.xfs /dev/rootvg/lvdata

[root@JCGY-postgres-2 ~]# mkfs.xfs /dev/rootvg/lvdata
meta-data=/dev/rootvg/lvdata     isize=512    agcount=4, agsize=12451840 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=49807360, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=24320, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


[root@JCGY-postgres-2 ~]# ls /
bin  boot  changepw.sh  dev  etc  home  lib  lib64  lock_root_sshlogin.sh  media  mklv.sh  mkvg.sh  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var



```







### 挂载方法2

```
parted -s /dev/sdb mklabel gpt
parted -s /dev/sdb mkpart primary 0 100%
mkfs.ext4 -T largefile /dev/vdb1 &
mkfs.ext4 -i 4096 /dev/vdb1
mkdir -p /data1
echo "/dev/vdb1 /data1 ext4 defaults 0 0" >> /etc/fstab
df -h
mount -a
df -h
reboot

mkfs.ext4 -T -i largefile /dev/vdb1 & 
mkfs.ext4 -i 4096 /dev/vdb1
```


### 方舟生产环境挂载

```


mkfs.ext4 -T largefile /dev/sdb2 &

parted -s /dev/sdc mklabel gpt
parted -s /dev/sdc mkpart primary 0 100%
mkfs.ext4 -T largefile /dev/sdc1 &
mkdir -p /data2
echo "/dev/vdb1 /data1 ext4 defaults 0 0" >> /etc/fstab
df -h
mount -a
df -h


```


### 格式部分磁盘

```
[sysadm@C11-1 ~]$ sudo su
[root@C11-1 sysadm]# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                       11:0    1  4.5G  0 rom  
sda                        8:0    0   64G  0 disk 
├─sda1                     8:1    0    1G  0 part /boot
└─sda2                     8:2    0   63G  0 part 
  ├─rootvg-lvroot (dm-0) 253:0    0   30G  0 lvm  /
  ├─rootvg-lvswap (dm-1) 253:1    0   12G  0 lvm  [SWAP]
  ├─rootvg-lvopt (dm-2)  253:2    0   10G  0 lvm  /opt
  ├─rootvg-lvvar (dm-3)  253:3    0   10G  0 lvm  /var
  └─rootvg-lvhome (dm-4) 253:4    0    1G  0 lvm  /home
sdc                        8:32   0  2.2T  0 disk 
sdd                        8:48   0  2.2T  0 disk 
sde                        8:64   0  2.2T  0 disk 
sdf                        8:80   0  2.2T  0 disk 
sdb                        8:16   0  2.2T  0 disk 
sdg                        8:96   0  2.2T  0 disk 
sdh                        8:112  0  2.2T  0 disk 
[root@C11-1 sysadm]# parted /dev/sdb
GNU Parted 2.1
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart                                                           
Partition name?  []? sdb1                                                 
File system type?  [ext2]? ext4
Start? 0                                                                  
End? 200G                                                                 
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? Ignore
(parted) p                                                                
Model: VMware Virtual disk (scsi)
Disk /dev/sdb: 2362GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End    Size   File system  Name  Flags
 1      17.4kB  200GB  200GB               sdb1

(parted) mkpart                                                           
Partition name?  []? sdb2                                                 
File system type?  [ext2]? ext4                                           
Start?                                                                    
Start?                                                                    
Start? 200G                                                               
End? 100%                                                                 
(parted) p                                                                
Model: VMware Virtual disk (scsi)
Disk /dev/sdb: 2362GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name  Flags
 1      17.4kB  200GB   200GB                sdb1
 2      200GB   2362GB  2162GB               sdb2

(parted) q                                                                
Information: You may need to update /etc/fstab.                           

[root@C11-1 sysadm]# lsblk
NAME                     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0                       11:0    1   4.5G  0 rom  
sda                        8:0    0    64G  0 disk 
├─sda1                     8:1    0     1G  0 part /boot
└─sda2                     8:2    0    63G  0 part 
  ├─rootvg-lvroot (dm-0) 253:0    0    30G  0 lvm  /
  ├─rootvg-lvswap (dm-1) 253:1    0    12G  0 lvm  [SWAP]
  ├─rootvg-lvopt (dm-2)  253:2    0    10G  0 lvm  /opt
  ├─rootvg-lvvar (dm-3)  253:3    0    10G  0 lvm  /var
  └─rootvg-lvhome (dm-4) 253:4    0     1G  0 lvm  /home
sdc                        8:32   0   2.2T  0 disk 
sdd                        8:48   0   2.2T  0 disk 
sde                        8:64   0   2.2T  0 disk 
sdf                        8:80   0   2.2T  0 disk 
sdb                        8:16   0   2.2T  0 disk 
├─sdb1                     8:17   0 186.3G  0 part 
└─sdb2                     8:18   0     2T  0 part 
sdg                        8:96   0   2.2T  0 disk 
sdh                        8:112  0   2.2T  0 disk 
[root@C11-1 sysadm]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root@C11-1 sysadm]# vgextend rootvg /dev/sdb1
  Volume group "rootvg" successfully extended
[root@C11-1 sysadm]# lvextend -L 50G /dev/rootvg/lvvar 
  Size of logical volume rootvg/lvvar changed from 10.00 GiB (2560 extents) to 50.00 GiB (12800 extents).
  Logical volume lvvar successfully resized.

[root@C11-1 sysadm]# resize2fs /dev/mapper/rootvg-lvvar 
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/mapper/rootvg-lvvar is mounted on /var; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 4
Performing an on-line resize of /dev/mapper/rootvg-lvvar to 13107200 (4k) blocks.
The filesystem on /dev/mapper/rootvg-lvvar is now 13107200 blocks long.

[root@C11-1 sysadm]# lsblk
NAME                     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0                       11:0    1   4.5G  0 rom  
sda                        8:0    0    64G  0 disk 
├─sda1                     8:1    0     1G  0 part /boot
└─sda2                     8:2    0    63G  0 part 
  ├─rootvg-lvroot (dm-0) 253:0    0    30G  0 lvm  /
  ├─rootvg-lvswap (dm-1) 253:1    0    12G  0 lvm  [SWAP]
  ├─rootvg-lvopt (dm-2)  253:2    0    10G  0 lvm  /opt
  ├─rootvg-lvvar (dm-3)  253:3    0    50G  0 lvm  /var
  └─rootvg-lvhome (dm-4) 253:4    0     1G  0 lvm  /home
sdc                        8:32   0   2.2T  0 disk 
sdd                        8:48   0   2.2T  0 disk 
sde                        8:64   0   2.2T  0 disk 
sdf                        8:80   0   2.2T  0 disk 
sdb                        8:16   0   2.2T  0 disk 
├─sdb1                     8:17   0 186.3G  0 part 
│ └─rootvg-lvvar (dm-3)  253:3    0    50G  0 lvm  /var
└─sdb2                     8:18   0     2T  0 part 
sdg                        8:96   0   2.2T  0 disk 
sdh                        8:112  0   2.2T  0 disk 
```

### 写入/etc/fastab

```
mkdir -p /data1
mkdir -p /data2
mkdir -p /data3
mkdir -p /data4
mkdir -p /data5
mkdir -p /data6
mkdir -p /data7

/dev/sdb2 /data1 ext4 defaults 0 0
/dev/sdc1 /data2 ext4 defaults 0 0
/dev/sdd1 /data3 ext4 defaults 0 0
/dev/sde1 /data4 ext4 defaults 0 0
/dev/sdf1 /data5 ext4 defaults 0 0
/dev/sdg1 /data6 ext4 defaults 0 0
/dev/sdh1 /data7 ext4 defaults 0 0
```





### 常见问题 

```
使用 resize2fs或xfs_growfs 对挂载目录在线扩容
resize2fs 针对文件系统ext2 ext3 ext4
xfs_growfs 针对文件系统xfs

### 报错 
[root@JL-C11-GYHLW-0129 ~]# xfs_growfs /dev/rootvg/lvvar
-bash: xfs_growfs: command not found
[root@JL-C11-GYHLW-0129 ~]# xfs_growfs /dev/rootvg/lvvar
-bash: xfs_growfs: command not found

### 使用resize2fs解决
[root@JL-C11-GYHLW-0129 ~]# resize2fs解决 /dev/mapper/rootvg-lvvar
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/mapper/rootvg-lvvar is mounted on /var; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 4
Performing an on-line resize of /dev/mapper/rootvg-lvvar to 13107200 (4k) blocks.

The filesystem on /dev/mapper/rootvg-lvvar is now 13107200 blocks long.
```



### 磁盘挂载全部

```
yum -y install vim

lsblk 查看磁盘，可以扩容
创建
pvcreate /dev/sdb 
pvs
vgcreate CDP_VG /dev/sdb
vgs  
lvcreate -l 100%VG -n CDP_LV CDP_VG 
lvs
lvdisplay
mkfs.xfs /dev/CDP_VG/CDP_LV
mkdir -p /data1 
vim /etc/fstab
echo '/dev/CDP_VG/CDP_LV  /data1  xfs     defaults        0 0'  >> /etc/fstab   && cat /etc/fstab   && mount -a  && df -h

/dev/mapper/CDP_VG-CDP_LV   50G   33M   50G   1% /data3
-------------------------------------------------------
fdisk -l ：/dev/sdd: 10.7 GB
lsblk：sdd  8:48   0   10G  0 disk

# 创建pv
pvcreate /dev/sdd
[root@docker1 ~]# pvcreate /dev/sdd
  Physical volume "/dev/sdd" successfully created.

# pvs验证
[root@docker1 ~]# pvs
  PV         VG     Fmt  Attr PSize    PFree  
  /dev/sda2  centos lvm2 a--   <49.00g      0 
  /dev/sdb   centos lvm2 a--  <300.00g <50.00g
  /dev/sdc   CDP_VG lvm2 a--   <50.00g      0 
  /dev/sdd          lvm2 ---    10.00g  10.00g
  
# 创建VG
- vgcreate CDP_VG(可修改) /dev/sdb（修改）
vgcreate VG /dev/sdd
[root@docker1 ~]# vgcreate VG /dev/sdd
  Volume group "VG" successfully created
  
# vgs验证
[root@docker1 ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree  
  CDP_VG   1   1   0 wz--n- <50.00g      0 
  VG       1   0   0 wz--n- <10.00g <10.00g
  centos   2   3   0 wz--n- 348.99g <50.00g

# 创建lv
-  lvcreate -l 100%VG -n CDP_LV（修改） CDP_VG(修改)
lvcreate -l 100%VG -n LV VG 
[root@docker1 ~]# lvcreate -l 100%VG -n LV VG
  Logical volume "LV" created.

# lvs&lvdisplay验证
[root@docker1 ~]# lvs
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  CDP_LV CDP_VG -wi-ao---- <50.00g                                                    
  LV     VG     -wi-a----- <10.00g                                                    
  lvdata centos -wi-ao---- 250.00g                                                    
  root   centos -wi-ao---- <47.00g                                                    
  var    centos -wi-ao----   2.00g

# 格式化
- mkfs.xfs /dev/VG/LV
root@docker1 ~]# mkfs.xfs /dev/VG/LV
meta-data=/dev/VG/LV             isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

# 挂载
mkdir -p /data4 
vim /etc/fstab
echo '/dev/VG/LV  /data4  xfs     defaults        0 0'  >> /etc/fstab   && cat /etc/fstab   && mount -a  && df -h
```

### 磁盘扩容
```
fdisk -l ：Disk /dev/sde: 10.7 GB
lsblk：sdc  8:32   0   10G  0 disk

# 创建pv
pvcreate /dev/sdc 
[root@docker1 ~]# pvcreate /dev/sdc 
  Physical volume "/dev/sdc" successfully created.

# pvs验证
[root@docker1 ~]#  pvs
  PV         VG     Fmt  Attr PSize    PFree  
  /dev/sda2  centos lvm2 a--   <49.00g      0 
  /dev/sdb   centos lvm2 a--  <300.00g <50.00g
  /dev/sdc          lvm2 ---    10.00g  10.00g
  /dev/sdd   CDP_VG lvm2 a--   <50.00g      0 
  /dev/sde   VG     lvm2 a--   <10.00g      0 
  
# 创建VG
- vgcreate CDP_VG(可修改) /dev/sdb（修改）
vgcreate VG /dev/sdd
[root@docker1 ~]# vgcreate VG /dev/sdd
  Volume group "VG" successfully created
  
# vgs验证
[root@docker1 ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree  
  CDP_VG   1   1   0 wz--n- <50.00g      0 
  VG       1   0   0 wz--n- <10.00g <10.00g
  centos   2   3   0 wz--n- 348.99g <50.00g

# 扩容
查看/dev/mapper/centos-root所在的VG Name为 centos
pvdisplay
PV Name               /dev/sde
VG Name               VG
------
- vgextend VG /dev/sdc  扩容
[root@docker1 ~]# vgextend VG /dev/sdc
  Volume group "VG" successfully extended

# 扩展
df -h
/dev/mapper/VG-LV           10G   33M   10G   1% /data4
- 添加所有空间
lvextend -l +100%FREE -r /dev/mapper/VG-LV
[root@docker1 ~]# lvextend -l +100%FREE -r /dev/mapper/VG-LV
  Size of logical volume VG/LV changed from <10.00 GiB (2559 extents) to 19.99 GiB (5118 extents).
  Logical volume VG/LV successfully resized.
meta-data=/dev/mapper/VG-LV      isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2620416 to 5240832

---------------------------------------
[root@CP-C11-T mysql]# lvextend -L 100G /dev/rootvg/lvvar 全部扩容给某块磁盘
[root@CP-C11-T mysql]# lvextend -L 100G /dev/rootvg/lvvar  扩容，lvextend -L +100G /dev/rootvg/lvvar #加100G
  Size of logical volume rootvg/lvvar changed from 2.00 GiB (512 extents) to 100.00 GiB (25600 extents).
  Logical volume rootvg/lvvar successfully resized.
[root@CP-C11-T mysql]# 
[root@CP-C11-T mysql]# xfs_growfs /dev/rootvg/lvvar 
meta-data=/dev/mapper/rootvg-lvvar isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 26214400
[root@CP-C11-T mysql]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/mapper/rootvg-lvroot   30G  6.3G   24G  21% /
devtmpfs                   7.9G     0  7.9G   0% /dev
tmpfs                      7.9G     0  7.9G   0% /dev/shm
tmpfs                      7.9G   41M  7.8G   1% /run
tmpfs                      7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/sda1                 1014M  160M  855M  16% /boot
/dev/mapper/rootvg-lvvar   100G  2.0G   98G   2% /var
/dev/mapper/rootvg-lvopt    10G  556M  9.5G   6% /opt
/dev/mapper/rootvg-lvhome 1014M   54M  961M   6% /home
tmpfs                      1.6G   12K  1.6G   1% /run/user/42
tmpfs  
```


### 磁盘做raid

```
Raid 0：一块硬盘或者以上就可做
raid0优势：数据读取写入最快，最大优势提高硬盘容量，比如3快80G的硬盘做raid0 可用总容量为240G。速度是一样。
缺点：无冗余能力，一块硬盘损坏，数据全无。
建议：做raid0 可以提供更好的容量以及性能，推荐对数据安全性要求不高的使用。


Raid 1：至少2快硬盘可做
raid1优势：镜像，数据安全强，2快硬盘做raid一块正常运行，另外一块镜像备份数据，保障数据的安全。一块坏了，另外一块硬盘也有完整的数据，保障运行。
缺点：性能提示不明显，做raid1之后硬盘使用率为50%.
建议：对数据安全性比较看着，性能没有太高要求的人使用。
---------------------------------------------------------------
Disk /dev/sde: 10.7 GB, 10737418240 bytes, 20971520 sectors

Disk /dev/sdd: 10.7 GB, 10737418240 bytes, 20971520 sectors

为DB机器准备数据文件路径/data1，首先创建raid，一般对ssd的盘做raid 0，这里sdb，sdc，sdd三块盘做raid0为例，具体的磁盘名称以实际的为准。
首先用lsblk查看磁盘信息
用mdadm工具做raid 0

mdadm -C /dev/md1 -a yes -l 1 -n 2 /dev/sdc /dev/sdd

[root@docker1 tmp]# mdadm -C /dev/md1 -a yes -l 1 -n 2 /dev/sdc /dev/sdd
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? 
Continue creating array? (y/n) 
Continue creating array? (y/n) y
mdadm: Fail to create md1 when using /sys/module/md_mod/parameters/new_array, fallback to creation via node
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
[root@docker1 tmp]# 

用mkfs工具将raid设备格式化为xfs文件系统
mkfs.xfs -f /dev/md1

mkdir -p /data5

手动mount挂载
mount /dev/md1 /data5

```

#### raid挂载失败解决办法
```
root@langchao-PC:/# mkdir  ${MOUNT_POINT} && mdadm -C ${RAID}  -a yes -l 1 -n 2 ${DEVICE1} ${DEVICE2}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: super1.x cannot open /dev/sdc: Device or resource busy
mdadm: /dev/sdc is not suitable for this array.
mdadm: create aborted

- 解决办法：
mdadm --stop /dev/md1
```
- http://www.manongjc.com/detail/51-jjagzwdpjzompiw.html

![image-20230825114021107](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825114021107.png)