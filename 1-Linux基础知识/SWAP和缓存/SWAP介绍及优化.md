# SWAP介绍及优化

# SWAP介绍

- Linux内核为了提高读写效率和速度，会将文件在内存中进行缓存，这部分内存就时cache memory（缓存内存），即使你的程序运行结束后，cache memory也不会自动释放，这就会导致在Linux系统中程序频繁的读写文件后，可用物理内存会变少，当系统的物理内存不够用的时候，就需要将物理内存中的一部分空间释放出来，以供当前运行的程序使用。那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到swap空间中，等到那些程序要运行时，再从swap分区中恢复保存的数据到内存中。这样系统总是在物理内存不够时，才进行swap交换。

# 查看swap分区大小

- 查看swap分区的大小以及使用情况，一般使用free -h 命令

  ```
  [root@localhost ~]# free -h
                total        used        free      shared  buff/cache   available
  Mem:           7.8G        259M        7.4G         22M        148M        7.3G
  Swap:          7.9G         29M        7.8G
  ```

# swap分区大小建议

- 系统的swap分区大小设置多大才是最优呢？下面是Redhat官方的文档中[关于swap分区大小的建议](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/installation_guide/s2-diskpartrecommend-ppc#id4394007)

  | 系统物理内存   | 建议的交换空间swap大小 | 如果允许休眠推荐的交换空间swap大小 |
  | -------------- | ---------------------- | ---------------------------------- |
  | ≤ 2 GB         | 物理内存容量的两倍     | 物理内存容量的三倍                 |
  | > 2 GB - 8 GB  | 与物理内存容量相等     | 物理内存容量的两倍                 |
  | > 8 GB - 64 GB | 至少 4 GB              | 物理内存容量的 1.5 倍              |
  | > 64 GB        | 至少 4 GB              | 不建议使用休眠                     |

- 在以上列出的每个范围临界点（使用2 GB、8 GB、或者64 GB物理内存的系统），可根据所选swap空间以及休眠支持自行裁决。如果系统资源允许此操作，增加swap空间可能会提高性能。可以在多个存储设备中分配swap空间，特别是对于那些使用高速驱动器、控制程序和接口的系统，还可以提高swap空间性能。

- 一般来说可以按照以下规则设置swap大小

  | 系统物理内存         | 建议设置                         |
  | -------------------- | -------------------------------- |
  | 4 GB以内的物理内存   | swap设置为内存的两倍，不超过4 GB |
  | 4 - 8 GB的物理内存   | swap等同于内存大小               |
  | 8 - 64 GB的物理内存  | swap 设置为 8 GB                 |
  | 64 GB 以上的物理内存 | swap 设置为 16 GB - 32 GB        |

# 手动添加swap分区





# SWAP优化

```
vim /etc/sysctl.conf
vm.swapiness=10
vm.dirty_ratio=10
vm.dirty_background_ratio=5
vm.dirty_expire_centisecs=500
vm.vfs_cache_pressure=500
```

## 各个配置的含义

### vm.swappiness

- swappiness的值的大小对如何使用swap分区是有着很大的联系的，swappiness=0的时候表示最大限度使用物理内存，然后才是swap空间，swappiness=100的时候表示积极的使用swap分区，并且把内存上的数据及时搬运到swap空间里面。使用如下命令查看

   ```
    [root@localhost ~]# cat /proc/sys/vm/swappiness
    30
   ```

- 30这个值也就是说，当你的内存在使用到100-30=70%的时候，就开始出现有交换分区的使用。内存的速度会比磁盘快很多，这样子会加大系统I/O，同时造成大量页的换进换出，严重影响系统的性能，所以哦我们在操作系统层面，要尽可能使用内存。

- 调整方法：

  - 临时调整

    ```
    [root@localhost ~]# sysctl -w vm.swappiness=10
    vm.swappiness = 10
    ```

  - 永久调整

    ```
    [root@localhost ~]# vim /etc/sysctl.conf
    ...
    vm.swappiness=10
    ```

### vm.dirty_ratio

同步刷脏页，会阻塞应用程序

- 这个参数控制文件系统的同步写缓冲区的大小，单位是百分比，表示当写缓冲使用到系统内存多少的时候（即指定了当文件系统缓存脏页达到系统内存百分之多少时，如10%），开始向磁盘写出数据，即系统不得不开始处理缓存脏页（因为此时脏页数量已经比较多，为了避免数据丢失需要将一定的脏页刷入外存），在此过程中很多应用进程可能会因为系统转而处理文件IO而阻塞。增大值会使用更多的系统内存用于磁盘写缓冲，也可以极大提高系统的写性能；但是，当你需要持续、恒定的写入场合时，应该降低其数值，一般启动上缺省值时10.

- 使用如下命令查看：

   ```
    [root@localhost ~]# cat /proc/sys/vm/dirty_ratio
    30
   ```

- 调整方法：

  - 临时调整

    ```
    sysctl -w vm.dirty_ratio=10
    vm.dirty_ratio = 10
    ```

  - 永久调整

    ```
    [root@localhost ~]# vim /etc/sysctl.conf
    ...
    vm.dirty_ratio=10
    ```

### vm.dirty_background_ratio

异步刷脏页，不会阻塞应用程序

- 这个参数控制文件系统的后台进程，在何时刷新磁盘。单位是百分比，表示系统内存的百分比，意思是当写缓冲使用到系统内存多少的时候，就会触发pdflush/flush/kdmflush等后台回写进程运行，将一定缓存的脏页异步地刷入外存。增大之会使用更多系统内存用于磁盘写缓冲，也可以极大提高系统的写性能。但是，当你需要持续、恒定的写入场合时，应该降低其数值，一般启动上缺省是 5。

- 注意：如果dirty_ratio设置比dirty_background_ratio大，可能认为dirty_ratio的触发条件不可能达到，因为每次肯定会先达到vm.dirty_background_ratio的条件，然而，确实是先达到vm.dirty_background_ratio的条件然后触发flush进程进行异步的回写操作，但是这一过程中应用进程仍然可以进行写操作，如果多个应用进程写入的量大于flush进程刷出的量那自然会达到vm.dirty_ratio这个参数所设定的坎，此时操作系统会转入同步地处理脏页的过程，阻塞应用进程。

- 使用如下命令查看：

   ```
    [root@localhost ~]# cat /proc/sys/vm/dirty_background_ratio
    10
   ```

- 调整方法

   - 临时调整

     ```
     [root@localhost ~]# sysctl -w vm.dirty_background_ratio=5
     vm.dirty_background_ratio = 5
     ```

   - 永久调整

     ```
     [root@localhost ~]# vim /etc/sysctl.conf
     ...
     vm.dirty_background_ratio=5
     ```

### vm.dirty_expire_centisecs

- 这个参数声明Linux内核写缓冲区里面的数据多“旧”了之后，pdflush进程就开始考虑写到磁盘中去。单位是 1/100秒。缺省是 3000，也就是 30 秒的数据就算旧了，将会刷新磁盘。对于特别重载的写操作来说，这个值适当缩小也是好的，但也不能缩小太多，因为缩小太多也会导致IO提高太快。建议设置为 1500，也就是15秒算旧。当然，如果你的系统内存比较大，并且写入模式是间歇式的，并且每次写入的数据不大（比如几十M），那么这个值还是大些的好。

- 使用如下命令查看：

  ```
  [root@localhost ~]# cat /proc/sys/vm/dirty_expire_centisecs
  3000
  ```

- 调整方法：

  - 临时调整

    ```
    sysctl -w vm.dirty_expire_centisecs=2000
    vm.dirty_expire_centisecs = 2000
    ```

  - 永久调整

    ```
    [root@localhost ~]# vim /etc/sysctl.conf
    ...
    vm.dirty_expire_centisecs=2000
    ```

### vm.dirty_writeback_centisecs

-  这个参数控制内核的脏数据刷新进程pdflush的运行间隔。单位是1/100秒。缺省数值是500，也就是 5 秒。如果你的系统是持续地写入动作，那么实际上还是降低这个数值比较好，这样可以把尖峰的写操作削平成多次写操作。如果你的系统是短期地尖峰式的写操作，并且写入数据不大（几十M/次）且内存有比较多富裕，那么应该增大此数值。

- 使用如下命令查看：

  ```
  [root@localhost ~]# cat /proc/sys/vm/dirty_writeback_centisecs
  500
  ```

- 调整方法：

  - 临时调整

    ```
    [root@localhost ~]# sysctl -w vm.dirty_writeback_centisecs=1000
    vm.dirty_writeback_centisecs = 1000
    ```

  - 永久调整

    ```
    [root@localhost ~]# vim /etc/sysctl.conf
    ...
    vm.dirty_writeback_centisecs=1000
    ```

### vm.vfs_cache_pressure

- 增大这个参数设置了虚拟内存回收directory和inode缓冲的倾向，这个值越大。越易回收。该文件表示内核回收用于directory和inode cache内存的倾向；缺省值100表示内核将根据pagecache和swapcache，把directory和inode cache保持在一个合理的百分比；降低该值低于100，将导致内核倾向于保留directory和inode cache；增加该值超过100，将导致内核倾向于回收directory和inode cache。

- 使用如下命令查看：

  ```
  [root@localhost ~]# cat /proc/sys/vm/vm.vfs_cache_pressure
  100
  ```

- 调整方法：

  - 临时调整

    ```
    [root@localhost ~]# sysctl -w vm.vfs_cache_pressure=500
    vm.vfs_cache_pressure = 500
    ```

  - 永久调整

    ```
    [root@localhost ~]# vim /etc/sysctl.conf
    ...
    vm.vfs_cache_pressure=500
    ```

    