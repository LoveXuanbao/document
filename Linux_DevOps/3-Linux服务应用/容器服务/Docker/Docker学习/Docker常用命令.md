# Docker查看容器进程在宿主机的PID

```bash
docker container top [containerID]
```

or

```bash
docker inspect -f '{{.State.Pid}}' [c]
```



# Docker容器日志清理脚本

```
```

# Docker查看容器hostport

```
docker inspect --format="{{json .HostConfig.PortBindings}}" [containerID]
```
