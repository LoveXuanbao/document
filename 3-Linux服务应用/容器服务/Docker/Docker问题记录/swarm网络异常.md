## 问题描述

docker swarm 启动的容器 REPLICAS 状态为 0/1，而docker service logs NAME 没有任何日志

![image-20230825145959801](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145959801.png)



## 排查过程

使用 `docker service scale redis=0` 命令把副本数改为0个，

然后在改回1，查看实时启动过程

![image-20230825150043838](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825150043838.png)

发现地址被占用



停掉我们的服务之后，查看network 网络还是被占用

```
docker network list
```

![image-20230825150135286](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825150135286.png)



删除该 network

```
docker network rm d9ef4e908bc1
```



重启服务，问题解决