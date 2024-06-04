####  kube-apiserver启动报错

##### 报错信息

```
kube-apiserver: Error: Unable to find suitable network address.error='no default routes found in "/proc/net/route" or "/proc/net/ipv6_route"'. Try to set the AdvertiseAddress directly or provide a valid BindAddress to fix this.
```

![image-20230825163957849](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163957849.png)



##### 解决方法

为服务器配置网关

```
# vi /etc/sysconfig/network-scripts/ifcfg-ens33
.....
GATEWAY=xxx.xxx.xxx.xxx
```