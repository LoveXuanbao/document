# 目标主机showmount -e信息泄露(CVE-1999-0554)



![image-20230825180249941](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825180249941.png)

# 解决办法

```bash
在NFS服务器上：
vi /etc/hosts.allow
mountd:192.168.1.1
rpcbind:192.168.1.1:allow

vi /etc/hosts.deny
mountd:ALL
rpcbind:ALL:deny
```

