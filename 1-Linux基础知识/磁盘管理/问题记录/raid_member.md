# linux_raid_member

## 问题背景

- 服务器操作系统重装，重装之后在初始化磁盘的时候报如下错误

```bash
/dev/sdb contains a linux_raid_member file system labelled 'file-System-Product-Name:0'
```

## 问题原因

- 因为之前服务器/dev/sdb这块磁盘做过软raid，重装操作系统之后，raid信息依然存在于/dev/sdb磁盘中

## 解决办法

- 先重写磁盘

```bash
# dd if=/dev/zero of=/dev/sdb bs=1M count=1
```

- 删除raid信息

```bash
# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md127 : inactive sdb1[1](S)
        7813893495 blocks super 1.2
         unused devices:
# mdadm -S /dev/md127
mdadm: stopped /dev/md127
```

- 重新格式化磁盘挂载即可