## 物理卷PV

### pvcreate

创建物理卷设备。在建立物理卷时，我们既可以把整块硬盘都建立成物理卷，也可以把某个分区建立成物理卷。

- 语法格式：

  **pvcreate [设备文件名]**

- 常用参数

  | 参数 | 说明                           |
  | ---- | ------------------------------ |
  | -f   | 强制创建物理卷，不需要用户确认 |
  | -u   | 指定设备的UUID                 |
  | -y   | 所有的问题都回答yes            |

```bash
# 指定一个磁盘设备创建
[root@localhost ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
# 指定多个磁盘设备分区创建
[root@localhost ~]# pvcreate /dev/sdc{1,2,3,4}
  Physical volume "/dev/sdc1" successfully created.
  Physical volume "/dev/sdc2" successfully created.
  Physical volume "/dev/sdc3" successfully created.
  Physical volume "/dev/sdc4" successfully created.
```

### pvscan

用来查询系统中哪些硬盘或分区是物理卷

- 语法格式：

  **pvscan [参数]**

- 常用参数

  | 参数 | 说明                         |
  | ---- | ---------------------------- |
  | -d   | 调试模式                     |
  | -e   | 仅显示属于输出卷组的物理卷   |
  | -n   | 仅显示不属于任何卷组的物理卷 |
  | -s   | 短格式输出                   |
  | -u   | 显示UUID                     |

```bash
[root@localhost ~]# pvscan
  PV /dev/sda2   VG centos          lvm2 [<499.00 GiB / 8.00 MiB free]
  Total: 1 [<499.00 GiB] / in use: 1 [<499.00 GiB] / in no VG: 0 [0   ]
[root@localhost ~]# pvscan -s
  /dev/sda2
  Total: 1 [<499.00 GiB] / in use: 1 [<499.00 GiB] / in no VG: 0 [0   ]
[root@localhost ~]# pvscan -u
  PV /dev/sda2 with UUID RvZ1RO-BGop-4Xke-0GU4-nbpL-21j7-wBQdT8    VG centos          lvm2 [<499.00 GiB / 8.00 MiB free]
  Total: 1 [<499.00 GiB] / in use: 1 [<499.00 GiB] / in no VG: 0 [0   ]
```

### pvdisplay

显示物理卷的属性。显示的物理卷信息包括：物理卷名称、所属的卷组、物理卷大小、PE大小，总PE数、可用PE数，已分配的PE数和UUID

- 语法格式：

  **pvdisplay [参数]**

- 常用参数

  | 参数 | 说明             |
  | ---- | ---------------- |
  | -s   | 以短格式输出     |
  | -m   | 显示PE到LE的映射 |

```bash
[root@localhost ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <499.00 GiB / not usable 3.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              127743
  Free PE               2
  Allocated PE          127741
  PV UUID               RvZ1RO-BGop-4Xke-0GU4-nbpL-21j7-wBQdT8
```

### pvremove

用于删除一个存在的物理卷，使用pvremove指令删除物理卷时，它将LVM分区上的物理卷信息删除，使其不再被视为一个物理卷

- 语法格式：

  **pvremove [参数]**

- 常用参数

  | 参数 | 说明                |
  | ---- | ------------------- |
  | -d   | 调试模式            |
  | -f   | 强制删除            |
  | -y   | 所有的问题都回答yes |

## 卷组VG

### vgcreate

用于创建卷组设备。

- 语法格式：

  **vgcreate 参数 卷组**

- 常用参数：

  | 参数 | 说明                         |
  | ---- | ---------------------------- |
  | -l   | 卷组上允许创建的最大逻辑卷数 |
  | -p   | 卷组中允许添加的最大物理卷数 |
  | -s   | 卷组上的物理卷的PE大小       |

```bash
[root@localhost ~]# vgcreate storage /dev/sdb /dev/sdc
 Volume group "storage" successfully created
```
