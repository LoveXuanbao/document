

要修改Docker Swarm中容器的shmsize大小，需要在Docker服务启动时设置shm-size参数。下面是具体步骤：

1. 在Docker Swarm集群中的每个节点上停止Docker服务：

```bash
sudo systemctl stop docker
```

1. 编辑Docker配置文件，通常为`/etc/docker/daemon.json`，如果没有则创建该文件，并添加以下内容：

```json
{
  "default-shm-size": "2G"
}
```

上述示例中设置了默认的shm-size为2GB，可以根据需要修改为其他大小。

1. 保存并退出编辑器，然后重新启动Docker服务：

```bash
sudo systemctl start docker
```

1. 在需要修改shm-size的容器启动时，指定shm-size参数：

```bash
docker run --shm-size=2g <image>
```