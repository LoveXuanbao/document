#### Service NodeManager failed in state STARTED; cause xxxx  : Corrupton: 22 missing file: 

- 报错如下：重启该节点nodemanager服务后报错任然存在

![image-20231214161701987](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231214161701987.png)

##### 解决办法：

1. 按照日志提示，备份当前节点上的 `/data2/var/log/hadoop-yarn/nodemanager/recovery-stte/` 目录下的 nm-aux-services 目录

```
cd /data2/var/log/hadoop-yarn/nodemanager/recovery-stte/
mv nm-aux-services nm-aux-services_bak_20231214
```

2. 重启NodeManager服务问题解决