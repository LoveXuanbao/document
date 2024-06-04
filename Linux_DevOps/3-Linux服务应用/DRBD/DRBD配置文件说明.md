# DRBD配置文件说明
- DRBD的主配置文件为/etc/drbd.conf；为了管理的便捷性，目前通常会将配置文件分成多个部分，且都保存在/etc/drbd.d目录中，主配置文件中仅使用"include"指令将这些配置文件片段整合起来。通常，/etc/drbd.d目录中的配置文件为global_common.conf和所有以.res结尾的文件。其中global_common.conf中主要定义global段和common段，而每一个.res的文件用于定义一个资源。

- 在配置文件中，global段仅能出现一次，且如果所有的配置信息都保存至用一个配置文件中而不分开多个文件的，global段必须位于配置文件的最开始处。目前global段中可以定义的参数仅有minor-count，dialog-refresh，disable-ip-verification和usage-count。

- common段则用于定义被每一个资源默认继承的参数，可以在资源定义中使用的参数都可以在common段中定义。实际应用中，common段并非必须，但建议将多个资源共享的参数定义为common段中的参数以降低配置文件的复杂度。

- resource段则用于定义DRBD资源，每个资源通常定义在一个单独的位于/etc/drbd.d目录中的以.res结尾的文件中。资源在定义时必须为其命名，名字可以由非空白的ASCII字符组成。每一个资源段的定义中至少要包含两个host字段，以定义此资源关联至的节点，其他参数均可以从common段或DRBD默认中进行继承而无须定义。


# DRBD配置文件
- common段是用来定义共享的资源参数，以减少资源定义的重复性。common段是非必须的。resource段一般为DRBd上每一个节点来定义其资源参数的。

```bash
[root@localhosts ~]# cat /etc/drbd.d/global_common.conf
global {
  usage-count yes;    ##是否参加DRBD使用者统计，默认是参加
  # minor-count dialog-refresh disable-ip-verification    //这里是global可以使用的参数
  # minor-count: 32    ##从(设备)个数，取值范围1~255，默认值为32。该选项设定了允许定义的resource个数，当要定义的resource超过了此选项的设定时，
                       ##需要重新载入DRBD内核模块。
  # disable-ip-verification: no    //是否禁用ip检查
}

common {
  protocol C;    ##指定复制协议，复制协议共有三种，为协议A，B，C，默认协议为C
  handlers {    ##该配置段用来定义一系列处理器，用来回应特定事件。
    # These are EXAMPLE handlers onlu.
    # They may have severe implications,
    # like hard resetting the node under certain circumstances.
    # Be careful when chosing your poison.
    # pri-on-incon-degr "/usr/lib/DRBD/notify-pri-on-incon-degr.sh; /usr/lib/DRBD/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
    # pri-lost-after-sb "/usr/lib/DRBD/notify-pri-lost-after-sb.sh; /usr/lib/DRBD/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
    # local-io-error "/usr/lib/DRBD/notify-io-error.sh; /usr/lib/DRBD/notify-emergency-shutdown.sh; echo o > /proc/sysrq-trigger ; halt -f";
    # fence-peer "/usr/lib/DRBD/crm-fence-peer.sh";
    # split-brain "/usr/lib/DRBD/notify-split-brain.sh root";
    # out-of-sync "/usr/lib/DRBD/notify-out-of-sync.sh root";
    # before-resync-target "/usr/lib/DRBD/snapshot-resync-target-lvm.sh -p 15 -- -c 16k";
    # after-resync-target /usr/lib/DRBD/unsnapshot-resync-target-lvm.sh;
  }
  startup {    ##DRBD同步时使用的验证方式和密码。该配置段用来更加精细地调节DRBD属性，它作用于配置节点在启动或重启时，常用选项有：
    # wfc-timeout degr-wfc-timeout outdated-wfc-timeout wait-after-sb
    wfc-timeout:    ##该选项设定一个时间值，单位是秒。在启动DRBD时，初始化脚本DRBD会阻塞启动进程的运行，直到对等节点的出现。该选项就是用来限制这个等待时间，
                    ##默认为0，即不限制，永远等待。
    degr-wfc-timeout:    ##该选项也设定一个时间值，单位是秒。也是用于限制等待时间，只是作用的情形不同，它作用于以及降级集群(即那些只剩下一个节点的集群)在重启时等待时间。
    outdated-wfc-timeout:    ##同上，也是用来设定等待时间，单位是秒。它用于设定等待过期节点的时间
  }
  disk {
    # on-io-error fencing use-bmby no-disk-barrier no-disk-flushes    ##这里是disk段内可以定义的参数
    # no-disk-drain no-md-flushes max-bio-bvecs    ##这里是disk段内可以定义的参数
    on-io-error: detach    ##选项：此选项设定了一个策略，如果底层设备向上层设备报告发生I/O错误，将按照该策略进行处理。有效策略包括：
      detach    ##发生I/O错误的节点将放弃底层设备，一diskless mode继续工作。在diskless mode下，只要还有网络连接，DRBD将从secondary node读写数据，
                ##而不需要failover(故障转移)。该策略会导致一定的损失，但好处也很明显，DRBD服务中断。官方推荐默认策略。
      pass_on    ##把I/O错误报告给上层设备。如果错误发生在primary节点，把它报告给文件系统，由上层设备处理这些错误（例如，它会导致文件系统以只读方式重新挂载），
                 ##它可能会导致DRBD停止提供服务；如果发生在secondary节点，则忽略该错误（因为secondary节点没有上层设备可以报告）。该策略曾经是默认策略，但现在已被detach所取代。
      call-local-io-error    ##调用预定义的本地local-io-error脚本进行处理。该策略需要在resource（或common）配置段的handlers部分，
                             ##预定义一个相应的local-io-error命令调用。该策略完全由管理员通过local-io-error命令（或脚本）调用来控制如何处理I/O错误。
    fencing:     ##该选项设定一个策略来避免split brain的状况。有效的策略包括：
      dont-care:     ##默认策略。不采取任何隔离措施。
      resource——only:     ##在此策略下，如果一个节点处于split brain状态，它将尝试隔离对端节点的磁盘。这个操作通过调用fence-peer处理器来实现。
                          ##fence-peer处理器将通过其他通信路径到达对等节点，并在对等节点上调用drbdadm outdate res命令。
      resource-and-stonith:     ##在此策略下，如果一个节点处于split brain状态，它将停止I/O操作，并调用fence-peer处理器。处理器其他通信路径到达对等节点，
                                ##并在这个对等节点上调用drbdadm outdate res命令。如果无法到达对等节点，它将向对等端发送关机命令。一旦问题解决，I/O操作将重新进行。
                                ##如果处理器失败，你可以使用resume-io命令来重新开始I/O操作。
  }
  net {     ##该配置段用来精细地调节DRBD属性，网络相关的属性。常用选项有：
    # sndbuf-size rcvbuf-size timeout connect-int ping-int ping-timeout max-buffers    ## 这里是net段内可以定义的参数
    # max-epoch-size ko-count allow-two-primaries cram-hmac-alg share-secret    ## 这里是net段内可以定义的参数
    # after-sb-0pri afer-sb-1pri after-sb-2pri data-integrity-alg no-tcp-cork    ## 这里是net段内可以定义的参数
    sndbuf-size:     ##该选项用来调节TCP send buffer的大小，DRBD8.2.7以前的版本，默认值为0，意味着自动调节大小；新版本的DRBD的默认值为128KiB。
                     ##高吞吐量网络(例如专用的千兆网卡或负载均衡中绑定连接)中，增加到512K比较合适，或者可以更高，但是最好不要超过2M。
    timeout:     ##该选项设定一个时间值，单位为0.1秒。如果搭档节点在此事件内发来应答包，那么就认为搭档节点已经死亡，因此将断开这次TCP/IP连接。
                 ##默认值为60，即6秒。该选项的值必须小于connnect-int和ping-int的值。
    connect-int:     ##如果无法立即连接上远程DRBD设备，系统将断续尝试连接。该选项设定的就是两次尝试间隔时间。单位为秒，默认值为10秒。
    ping-timeout:     ##该选项设定一个时间值，单位是0.1秒。如果对端节点没有在此事件内应答keep-alive包，它将被认为已经死亡。默认值为300ms。
    max-buffers:     ##该选项设定一个由DRBD分配的最大请求数，单位是页面大小(PAGE_SIZE)，大多数系统中，页面大小为4KB。这些buffer用来存储那些即将写入磁盘的。
                     ##最小值为32(即128KB)。这个值大一点好。
    max-epoch-size：    ##该选项设定了两次write barriers之间最大的数据块数。如果选项的值小于10，将影响系统性能。大一点好
    ko-count：     ##该选项设定一个值，把该选项设定的值 乘以 timeout设定的值，得到一个数字N，如果secondary节点没有在此时间内完成单次写请求，
                   ##它将从集群中被移除（即，primary node进入StandAlong模式）。取值范围0~200，默认值为0，即禁用该功能。
    allow-two-primaries：   ##这个是DRBD8.0及以后版本才支持的新特性，允许一个集群中有两个primary node。该模式需要特定文件系统的支撑，
                            ##目前只有OCFS2和GFS可以，传统的ext3、ext4、xfs等都不行！
    cram-hmac-alg：    ##该选项可以用来指定HMAC算法来启用对端节点授权。DRBD强烈建议启用对端点授权机制。可以指定/proc/crypto文件中识别的任一算法。
                       ##必须在此指定算法，以明确启用对端节点授权机制，实现数据加密传输。
    shared-secret：    ##该选项用来设定在对端节点授权中使用的密码，最长64个字符。
    data-integrity-alg：   ##该选项设定内核支持的一个算法，用于网络上的用户数据的一致性校验。通常的数据一致性校验，由TCP/IP头中所包含的16位校验和来进行，
                           ##而该选项可以使用内核所支持的任一算法。该功能默认关闭。
  }
  syncer {       ##该配置段用来更加精细地调节服务的同步进程。常用选项有
  # rate after al-extents use-rle cpu-mask verify-alg csums-alg
  rate：    ##设置同步时的速率，默认为250KB。默认的单位是KB/sec，也允许使用K、M和G，如40M。注意：syncer中的速率是以bytes，而不是bits来设定的。
            ##配置文件中的这个选项设置的速率是永久性的，但可使用下列命令临时地改变rate的值：DRBDsetup /dev/DRBDN syncer -r 100M。
            ##如果想重新恢复成drbd.conf配置文件中设定的速率，执行如下命令： DRBDadm adjust resource
  verify-alg：    ##该选项指定一个用于在线校验的算法，内核一般都会支持md5、sha1和crc32c校验算法。在线校验默认关闭，必须在此选项设定参数，以明确启用在线设备校验。
                  ##DRBD支持在线设备校验，它以一种高效的方式对不同节点的数据进行一致性校验。在线校验会影响CPU负载和使用，但影响比较轻微。
                  ##DRBD 8.2.5及以后版本支持此功能。一旦启用了该功能，你就可以使用下列命令进行一个在线校验： DRBDadm verify resource。
                  ##该命令对指定的resource进行检验，如果检测到有数据块没有同步，它会标记这些块，并往内核日志中写入一条信息。这个过程不会影响正在使用该设备的程序。
                  ## 如果检测到未同步的块，当检验结束后，你就可以如下命令重新同步它们：DRBDadm disconnect resource   or   DRBDadm connetc resource
 }
}
```

# 资源配置文件
```bash
[root@localhost ~]# cat /etc/drbd.d/web.res
resource web {    ##web为资源名称，可以自己指定
 on ha1.xsl.com {    ##on后面为节点的名称,有几个节点就有几个on段，这里是定义节点ha1.xsl.com上的资源
  device   /dev/DRBD0;       ##定义DRBD虚拟块设备，这个设备事先不要格式化。
  disk /dev/sda6;        ##定义存储磁盘为/dev/sda6,该分区创建完成之后就行了，不要进行格式化操作
  address 192.168.108.199:7789;      ##定义DRBD监听的地址和端口，以便和对端进行通信
  meta-disk  internal;       ##该参数有2个选项：internal和externally，其中internal表示将元数据和数据存储在同一个磁盘上；
                             ##而externally表示将元数据和数据分开存储，元数据被放在另一个磁盘上。
 }
 on ha2.xsl.com {        ##这里是定义节点ha2.xsl.com上的资源
  device /dev/DRBD0;
  disk /dev/sda6;
  address 192.168.108.201:7789;
  meta-disk internal;
 }
}
```