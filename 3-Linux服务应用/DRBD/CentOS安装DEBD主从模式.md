# 一、服务器信息（Centos7.4）

```
192.168.1.100    主服务器    主机名：primary
192.168.1.101    备服务器    主机名：secondary
```

# 二、两台服务器的防火墙都要互相允许访问。

- 最好时关闭selinux和firewalld防火墙（两台机器同样操作）

```
[root@Primary ~]# setenforce 0             //临时性关闭；永久关闭的话，需要修改/etc/sysconfig/selinux的SELINUX为disabled
[root@Primary ~]# systemctl stop firewalld.service    ##关闭防火墙
[root@Primary ~]# systemctl disable firewalld.service    ##关闭防火墙开机自启动
```

# 三、设置hosts文件

- 两台机器同样操作

```
[root@Primary ~]# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.101 Primary
192.168.1.102 Secondary
```

# 四、两台机器时钟同步

```
[root@Primary ~]# yum install -y netp    ##注意：此处安装完ntp服务后，需要修改ntp配置文件，此处不在标明怎么修改
[root@Primary ~]# 
[root@Primary ~]# ntpdate -u [serverIP]
```

# 五、升级Linux系统内核

- 两台机器同样操作
- 原装的Centos 7.4的内核不支持drbd，所以需要升级内核版本

```
[root@Primary ~]# rpm -Uvh kernel-ml-4.15.6-1.el7.elrepo.x86_64.rpm
warning: kernel-ml-4.15.6-1.el7.elrepo.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID baadae52: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:kernel-ml-4.15.6-1.el7.elrepo    ################################# [100%]

[root@Primary ~]# grub2-set-default 0     ##指定以新装的内核启动
[root@Primary ~]# reboot    ##重启生效
```

# 六、DRBD的安装

- 两台机器上同样操作
- 这里采用rpm方式安装

```
[root@Primary]# rpm -ivh drbd90-utils-9.6.0-1.el7.elrepo.x86_64.rpm
warning: drbd90-utils-9.6.0-1.el7.elrepo.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID baadae52: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:drbd90-utils-9.6.0-1.el7.elrepo  ################################# [100%]

[root@Primary ~]# modprobe drbd     加载模块
[root@Primary ~]# lsmod | grep drbd     ##查看模块是否加载上
drbd                  376832  0
lru_cache              16384  1 drbd
libcrc32c              16384  2 xfs,drbd
```

# 七、修改配置

- 两台机器同样操作

- drbd.conf文件配置如下，一般该文件时不需要修改的

```
[root@Primary ~]# cat /etc/drbd.conf
# You can find an example in  /usr/share/doc/drbd.../drbd.conf.example

include "drbd.d/global_common.conf";
include "drbd.d/*.res";
```

- global_common.conf配置文件如下

```
[root@Primary ~]# cp /etc/drbd.d/global_common.conf /etc/drbd.d/global_common.conf.bak
[root@Primary ~]# vim /etc/drbd.d/global_common.conf
global {
  usage-count yes;
}

common {
  protocol C;

  handlers {
  }

  startup {
        wfc-timeout          300;
        degr-wfc-timeout     300;
        outdated-wfc-timeout 300;
  }

  disk {
        on-io-error detach;
        no-disk-flushes;
        no-disk-barrier;
        c-plan-ahead 80;
        c-fill-target 192M;
        c-min-rate 80M;
        c-max-rate 800M;
  }

  net {
        cram-hmac-alg md5;
        shared-secret "testdrbd";
        max-buffers             72k;
        sndbuf-size            8192k;
        rcvbuf-size            10000k;
  }

  syncer {
        rate 500M;
  }
}
```

- r0.res资源配置文件如下

```
[root@Primary ~]# vim /etc/drbd.d/r0.res
resource r0 {        ## r0是资源名，可以不修改
on Primary {             ## primary,是主节点主机名
  device     /dev/drbd0;           ## 这是Primary机器上的DRBD虚拟块设备，事先不要格式化
  disk       /dev/centos/lvnfs;            ## 需要做成drbd的设备，可以是一块磁盘、一个分区或lv逻辑卷
  address    192.168.1.101:7898;
  meta-disk  internal;
 }
on Secondary {       ## secondary，是备节点主机名
  device     /dev/drbd0;          ## 这是Secondary机器上的DRBD虚拟块设备，事先不要格式化
  disk       /dev/centos/lvnfs;           ## 需要做成drbd的设备，可以是一块磁盘、一个分区或lv逻辑卷
  address    192.168.1.102:7898;　　　　　## DRBD监听的地址和端口。端口可以自己定义
  meta-disk  internal;
 }
}
```

# 八、在两台机器上添加lv分区，不要格式化

- 在两台机器上创建添加一个磁盘，并加入到vg里，创建10个G的lv分区给DRBD使用，分区为`/dev/centos/lvnfs`，不做格式化，并在本地系统创建`/data`目录，用户挂载，但是不做挂载操作

```
[root@Primary ~]# lsblk    ##查看空闲磁盘
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb               8:16   0   20G  0 disk
sr0              11:0    1  4.2G  0 rom
sda               8:0    0   20G  0 disk
├─sda2            8:2    0   19G  0 part
│ ├─centos-swap 253:1    0    2G  0 lvm  [SWAP]
│ └─centos-root 253:0    0   17G  0 lvm  /
└─sda1            8:1    0    1G  0 part /boot
[root@Primary ~]# pvcreate /dev/sdb     ##把sdb加入到pv里
  Physical volume "/dev/sdb" successfully created.
[root@Primary ~]# vgs    ##查看vg信息
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <19.00g    0
[root@Primary ~]# vgextend centos /dev/sdb      ##把sdb加入到vg里
  Volume group "centos" successfully extended
[root@Primary ~]# lvcreate -L 10G -n lvnfs centos
  Logical volume "lvnfs" created.
[root@Primary ~]# mkdir /data     ##创建/data目录
```

# 九、在两台设备上分别创建DRBD设备并激活r0资源

- 两台设备都要操作

```
[root@Primary ~]# drbdadm create-md r0
initializing activity log
initializing bitmap (320 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.
```

# 十、启用drbd服务

- 注意：需要主从共同启动才能生效

```
[root@Primary ~]# systemctl start drbd
[root@Primary ~]# cat /proc/drbd     ##查看DRBD撞他i
version: 8.4.10 (api:1/proto:86-101)
srcversion: 17A0C3A0AF9492ED4B9A418
 0: cs:Connected ro:Secondary/Secondary ds:Inconsistent/Inconsistent C r-----
    ns:0 nr:0 dw:0 dr:0 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:d oos:10485404
```

> 由上面两台主机的状态查看结果表示
>
> ro:Secondary/Secondary：表示两台主机的状态都是备节点。
>
> ds:Inconsistent/Inconsistent：表示磁盘状态，Inconsistent表示不一致，因为DRBD无法判断哪一台为主机，应该以哪一台的磁盘数据作为标准。

# 十一、将Primary主机配置为DRBD的主节点

```
[root@Primary ~]# drbdsetup /dev/drbd0 primary --force
[root@Primary ~]# cat /proc/drbd
version: 8.4.10 (api:1/proto:86-101)
srcversion: 17A0C3A0AF9492ED4B9A418
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:714752 nr:0 dw:0 dr:860232 al:8 bm:0 lo:0 pe:1 ua:149 ap:0 ep:1 wo:d oos:9770652
        [>...................] sync'ed:  6.9% (9540/10236)M     ##这里可以看到两台主机正在同步元数据
        finish: 0:00:54 speed: 178,688 (178,688) K/sec    ##预计需要的时间，及同步速度

[root@Primary ~]# cat /proc/drbd
version: 8.4.10 (api:1/proto:86-101)
srcversion: 17A0C3A0AF9492ED4B9A418
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:10485404 nr:0 dw:0 dr:10487524 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:d oos:0
```

> ro在主从服务器上分别显示：Priamry/Secondary和Secondary/Primay
>
> ds显示UpToDate/UpToDate 表示主从配置成功

# 十二、格式化改在DRBD（Primary主节点操作）

- 先格式化/dev/drbd0，格式化成xfs格式的文件系统

```
[root@Primary ~]# mkfs.xfs /dev/drbd0
meta-data=/dev/drbd0             isize=512    agcount=4, agsize=655338 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2621351, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- 创建挂载目录/data，然后执行DRBD挂载

```
[root@Primary ~]# mkdir /data
[root@Primary ~]# mount /dev/drbd0 /data
[root@Primary ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 470M     0  470M   0% /dev
tmpfs                    482M     0  482M   0% /dev/shm
tmpfs                    482M  6.7M  476M   2% /run
tmpfs                    482M     0  482M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.3G   16G   8% /
/dev/sda1               1014M  155M  860M  16% /boot
tmpfs                     97M     0   97M   0% /run/user/0
/dev/drbd0                10G   33M   10G   1% /data        ##可以看到drbd已经挂载成功
```

> 特别注意：Secondary节点上不允许对DRBD设备进行任何操作，包括只读，所有的读写操作都只能在Primary节点上进行。只有当Primary节点挂掉时，Secondary节点才能提升为Primary节点。

# 十三、DRBD故障模拟切换测试

- 模拟Primary节点发生故障，Secondary接管并提升为Primary
- 下面时在Primary节点上操作记录：进入挂载目录，创建两个测试文件，然后退出目录，卸载DRBD，然后把节点状态切换为Secondary，然后查看是否切换成功。

```
[root@Primary ~]# cd /data/
[root@Primary data]# touch test{1..2}.txt
[root@Primary data]# ll
total 0
-rw-r--r-- 1 root root 0 Oct 17 21:33 test1.txt
-rw-r--r-- 1 root root 0 Oct 17 21:33 test2.txt
[root@Primary data]# cd
[root@Primary ~]# umount /dev/drbd0
[root@Primary ~]# drbdsetup /dev/drbd0 secondary
[root@Primary ~]# cat /proc/drbd
```

- 下面实在Secondary节点上操作记录：把Secondary节点上的主机提升为Primary节点，查看是否切换工程，成功之后创建挂载目录，把DRBD挂载到/data目录下，然后查看刚才在Primary节点创建的文件是否存在。存在即成功。

```
[root@Secondary drbd.d]# drbdsetup /dev/drbd0 primary
[root@Secondary drbd.d]# cat /proc/drbd
version: 8.4.10 (api:1/proto:86-101)
srcversion: 17A0C3A0AF9492ED4B9A418
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:0 nr:12663 dw:12663 dr:2120 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:d oos:0
[root@Secondary drbd.d]# mkdir /data
[root@Secondary drbd.d]# mount /dev/drbd0 /data
[root@Secondary drbd.d]# ll /data
total 0
-rw-r--r-- 1 root root 0 Oct 17 21:33 test1.txt
-rw-r--r-- 1 root root 0 Oct 17 21:33 test2.txt
```
