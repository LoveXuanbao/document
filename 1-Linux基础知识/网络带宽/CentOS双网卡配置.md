# Centos配置双网卡同时访问路由

> `Vmware`创建了两台服务器，服务器有两张网卡分别是 `ens33`和`ens34`，其中`ens33`配置的仅主机模式，内网IP为：192.168.88.100/24；`ens34`配置nat模式，IP为DHCP自动获取。

## VMware配置

![image-20230825115422006](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825115422006.png)



### 一、配置ens33网卡（内网）

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=be0dfb84-a925-4133-af95-d00888c32ba9
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.88.100
NEWMASK=255.255.255.0
```

### 二、配置ens34网卡（外网）

> 只需要添加dns的配置，其余保持默认即可

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens34
UUID=ea8ea71b-89da-4283-b1e5-5c559de94d4a
DEVICE=ens34
ONBOOT=yes
DNS1=114.114.114.114
DNS2=8.8.8.8
```

### 三、添加内网访问路由

```
# vi /etc/sysconfig/network-scripts/route-ens33
ADDRESS=192.168.88.100    ## 内网IP
NETMASK=255.255.255.0     ## 内网IP的掩码
GATEWAY=192.168.88.254    ## 内网网关
```

### 四、重启网卡服务

```
# systemctl restart network
```

### 五、ping百度测试

```
[root@localhost ~]# ping www.baidu.com
PING www.a.shifen.com (182.61.200.6) 56(84) bytes of data.
64 bytes from 182.61.200.6 (182.61.200.6): icmp_seq=1 ttl=128 time=5.76 ms
64 bytes from 182.61.200.6 (182.61.200.6): icmp_seq=2 ttl=128 time=6.90 ms
64 bytes from 182.61.200.6 (182.61.200.6): icmp_seq=3 ttl=128 time=10.7 ms
64 bytes from 182.61.200.6 (182.61.200.6): icmp_seq=4 ttl=128 time=5.75 ms
^C
--- www.a.shifen.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3013ms
rtt min/avg/max/mdev = 5.750/7.293/10.758/2.056 ms
```