

### zookeeper

zookeeper启动正常启动显示成功，但是进程不在

日志显示报错：`Unable to load database on disk`

![image-20231109161349924](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231109161349924.png)

### 解决办法

- 找到 zoo.cfg 配置文件中的 dataDir 配置的路径，然后备份 version-2 目录，重建创建 version-2 目录（创建后注意权限问题），再次重启zk即可