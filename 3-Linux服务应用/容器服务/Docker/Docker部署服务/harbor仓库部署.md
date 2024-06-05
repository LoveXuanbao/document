## 环境信息

| 操作系统   | 主机名 | IP地址        |
| ---------- | ------ | ------------- |
| CentOS 7.4 | harbor | 192.168.44.69 |

## Harbor安装部署

- harbor以容器的形式进行部署，因此可以被部署到任何支持Docker的Linux发行版，需要安装docker和docker-compose编排工具，并具备如下环境：

  - **Python 2.7+**
  - **Docker Engine 1.10+**

  - **Docker Compose 1.6.0+**

- **docker 18.09.1** 以及以上版本，系统内核需要升级到4.4.x
  - CentOS 7.x 系统自带的 3.10.x 内核存在一些BUG，导致运行的Dokcer、kubernetes不稳定；
  - 搞版本的Docker（1.13以后）启用了3.10 kernel 实验支持的 **kernel memory account**功能（无法关系），当Docker节点压力大（如频繁的启动和停止容器）时，会导致**cgroup memory leak**;
  - Docker 19.09.1 及以上版本，需要手动升级内核到 **4.4.x**以上，否则会出现，harbor部署完成，正常启动，端口正常监听，防火墙也关闭，但是通过**http://IP:80**访问不了harbor，并且**/var/log/harbor**目录下没有任何日志产生

### 升级内核版本

- rpm 安装

```
[root@harbor tmp]# rpm -Uvh kernel-ml-5.2.14-1.el7.elrepo.x86_64.rpm 
warning: kernel-ml-5.2.14-1.el7.elrepo.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID baadae52: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:kernel-ml-5.2.14-1.el7.elrepo    ################################# [100%]
```

- 执行 **grub2-set-default 0** 设置从新内核启动

```
[root@harbor tmp]# grub2-set-default 0
```

- 执行 **grub2-mkconfig -o /boot/grub2/grub.cfg** 重新创建内核配置

```
[root@harbor tmp]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.2.14-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.2.14-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-479c296aa4504d1fab43f5789550e4ef
Found initrd image: /boot/initramfs-0-rescue-479c296aa4504d1fab43f5789550e4ef.img
Found linux image: /boot/vmlinuz-0-rescue-6b1dd57fc2909f65a4da9a33283d86f4
Found initrd image: /boot/initramfs-0-rescue-6b1dd57fc2909f65a4da9a33283d86f4.img
done
```

### 安装Docker

- yum 安装以下四个rpm 包

```
docker-ce-18.09.8-3.el7.x86_64.rpm
docker-ce-cli-18.09.8-3.el7.x86_64.rpm
container-selinux-2.107-3.el7.noarch.rpm
containerd.io-1.2.2-3.el7.x86_64.rpm
```

- 修改 docker 数据目录，找到 ExecStart ，在该行后面添加 **--graph=/data/docker** ，最好是空间比较大的磁盘

```
[root@harbor data]# vim /usr/lib/systemd/system/docker.service
...
[Service]
...
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --graph=/data/docker
...
```

- 重新加载配置文件，并启动 docker

```
[root@harbor data]# systemctl daemon-reload
[root@harbor data]# systemctl restart docker
```

- 创建 harbor 存储目录

```
[root@harbor data]# mkdir /data/harbor
```

- 拷贝 docker-compose 文件到 /usr/sbin/ 目录下，并赋予 755 权限

```
[root@harbor tmp]# cp docker-compose /usr/sbin
[root@harbor tmp]# chmod 755 /usr/sbin/docker-compose
[root@harbor tmp]# ll /usr/sbin/docker-compose 
-rwxr-xr-x 1 root root 16168192 Jan  5 14:35 /usr/sbin/docker-compose
```

- 解压 harbor.tar.gz

```
[root@harbor tmp]# tar -zxvf harbor.tar.gz 
harbor/
harbor/LICENSE
harbor/install.sh
harbor/prepare
harbor/harbor.yml
```

- 进入到 harbor 目录备份原配置文件，并修改

```
[root@harbor tmp]# cd harbor
[root@harbor-node harbor]# cp harbor.yml harbor.yml.bak
[root@harbor-node harbor]# vim harbor.yml
hostname: docker.harbor.com   ## harbor自身的ip或者主机名
http:
  port: 80
harbor_admin_password: admin123   ## harbor初始管理员密码
database:
  password: root123    ## postgres数据库的root密码
  max_idle_conns: 50
  max_open_conns: 100
data_volume: /data/harbor   ## harbor的数据存储目录
```

- 修改 prepare 文件，第 55 行， prepare 镜像的地址

```
[root@harbor tmp]# vim harbor/prepare
 docker run --rm -v $input_dir:/input:z \
                      -v $data_path:/data:z \
                      -v $harbor_prepare_path:/compose_location:z \
                      -v $config_dir:/config:z \
                      -v $secret_dir:/secret:z \
                      -v /:/hostfs:z \
                      repository.test.com:8444/repository/test/moebius/goharbor/prepare:v1.9.3 $@
```

- 执行 **./prepare** 安装 harbor

```
[root@harbor harbor]# ./prepare 
prepare base dir is set to /tmp/harbor
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /secret/keys/secretkey
Generated certificate, key file: /secret/core/private_key.pem, cert file: /secret/registry/root.crt
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
```

- 安装完后后，会发现解压目录harbor下面多个了一个docker-compose.yml文件，里边包含了harbor依赖的镜像和对应容器的创建信息
- 修改docker-compose.yml文件中的image镜像地址

```
[root@harbor tmp]# cat harbor/docker-compose.yml  | grep image
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-log:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/registry-photon:v2.7.1-patch-2819-2553-v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-registryctl:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-db:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-core:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-portal:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/harbor-jobservice:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/redis-photon:v1.9.3
    image: repository.test.com:8444/repository/test/moebius/goharbor/nginx-photon:v1.9.3
```

- 执行 **docker-compose up -d**     **注意：**需要在harbor解压完的目录下执行

```
[root@harbor harbor]# docker-compose up -d
Creating harbor-log ... done
Creating harbor-portal ... done
Creating registryctl   ... done
Creating redis         ... done
Creating registry      ... done
Creating harbor-db     ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
```

- 查看服务状态

```
[root@harbor harbor]# docker-compose ps
      Name                     Command                  State                 Ports          
---------------------------------------------------------------------------------------------
harbor-core         /harbor/harbor_core              Up (healthy)                            
harbor-db           /docker-entrypoint.sh            Up (healthy)   5432/tcp                 
harbor-jobservice   /harbor/harbor_jobservice  ...   Up (healthy)                            
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up (healthy)   8080/tcp                 
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:80->8080/tcp     
redis               redis-server /etc/redis.conf     Up (healthy)   6379/tcp                 
registry            /entrypoint.sh /etc/regist ...   Up (healthy)   5000/tcp                 
registryctl         /harbor/start.sh                 Up (healthy)
```

- 浏览器页面访问，使用默认账号**admin/admin123**登录

![image-20230825145303453](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145303453.png)

![image-20230825145355292](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145355292.png)

## 部署完成...