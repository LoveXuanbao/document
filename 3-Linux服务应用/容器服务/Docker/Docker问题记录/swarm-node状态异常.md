## docker swarm node 状态异常

客户生产环境服务访问不了，排查中发现 docker swarm node  STATUS 为 Down，特此记录解决过程

![image-20230825145547114](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145547114.png)



### 解决过程

根据[官方文档](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-losing-the-quorum)尝试新建集群

1、尝试移除集群，不成功

```
docker swarm leave    ## 移除集群命令
docker swarm leave --force    ## 根据上一条命令提示，添加 --force参数
```

![image-20230825145646763](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145646763.png)





2、尝试找到swarm数据文件，发现该文件异常的大。说明此数据是有问题，停止 docker 服务，备份该文件，重启docker服务，重新查看node状态，已经恢复正常

```
find / -name tasks.db
```

![image-20230825145734973](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145734973.png)



```
systemctl stop docker
mv tasks.db tasks.db.bak
systemctl start docker
```

![image-20230825145849850](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145849850.png)