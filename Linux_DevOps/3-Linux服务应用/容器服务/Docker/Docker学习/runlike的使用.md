# 介绍

`runlike` 是一个命令行工具，用于将 `Docker`容器的配置信息转换为人类可读的格式。它可以将 `Docker` 容器的命令行参数、环境变量、挂载点等信息转换为易于理解的格式，以便于容器的管理和维护

## github项目地址

```bash
https://github.com/lavie/runlike/blob/master/README.md
```

## docker镜像拉取地址

```bash
docker pull assaflavie/runlike
```

# 安装

## 方式一

使用 `pip` 方式安装

```bash
pip install runlike
```

## 方式二

免安装方式，使用 `docker `镜像来启动

1. 首先拉取镜像到要安装的服务器

```bash
docker pull assaflavie/runlike
```

2. 通过别名的方式运行 runlike，也可以把命令添加到 `~/.profile` 或者 `~/.bashrc` 来运行

```bash
alias runlike="docker run --rm -v /var/run/docker.sock:/var/run/docker.sock assaflavie/runlike:latest"
```

# 使用

`runlike -p 容器名/容器ID`

```bash
[root@localhost ~]# runlike -p gogs
docker run \
        --name=gogs \
        --hostname=3d892e4483b2 \
        --mac-address=02:42:ac:11:00:02 \
        --env=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
        --env=GOGS_CUSTOM=/data/gogs \
        --volume=/var/gogs:/data/gogs_data \
        --volume=/backup \
        --volume=/data \
        --workdir=/app/gogs \
        -p 10022:22 \
        -p 10888:3000 \
        --restart=no \
        --runtime=runc \
        --detach=true \
        -t \
        repository.test.com:8444/kubernetes/gogs/gogs:0.14.0 \
        /bin/s6-svscan /app/gogs/docker/s6/
```

