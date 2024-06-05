# 前言

## Rsync + inotify

`Rsync`与传统的`cp`、`tar`备份方式相比，`Rsync`具有安全性高、备份迅速、支持增量备份等优点，通过`rsync`可以解决对实时性要求不高的数据备份需求，例如定期的备份文件服务器数据到远端服务器，对本地磁盘定期做数据镜像等。

随着应用系统规模的不断扩大，对数据的安全性和可靠性也提出了更好的要求，`rsync`在高端业务系统中也逐渐暴漏出了许多不足。首先，`rsync`同步数据时，需要扫描所有文件后进行对比，进行差量传输。如果文件数量达到了百万甚至千万量级，扫描所有文件是非常耗时的。而且正在发生变化的往往是其中很少的一部分，这是非常低效的方式来。其次，`rsync`不能实时的去监测，同步数据，虽然可以通过 Linux 守护进程的方式进行触发同步，但是两次触发动作一定会有时间差，这样就导致了服务端和客户端数据可能出现不一致，无法在应用故障时完全的恢复数据。基于以上因素，`rsync + inotify`组合出现了！

`Inotify`是一种强大的、细粒度的、异步的文件系统事件监控机制，Linux 内核从`2.6.13`起，加入了`Inotify`支持，通过`Inotify`可以监控文件系统中添加、删除、修改、移动等各种细微事件，利用这个内核接口，第三方软件就可以监控文件系统下文件的各种变化情况，而`inotify-tools`就是这样的一个第三方软件。

`Rsync`可以实现触发式的文件同步，但是通过 `crontab` 守护进程方式进行触发，同步的数据和实际数据会有差异，而 `inotify`可以监控文件系统的各种变化，当文件又任何变动时，就触发 `rsync`同步，这样就刚好解决了同步数据的实时性问题。

# 一、准备工作

- windows上安装`ubuntu`子系统
- Linux发行版：`CentOS 7.8`

# 二、Centos 7.8安装Rsync服务

一般来说，在安装完 `Centos 7.8` 操作系统后，默认都带有 `rsync` 服务

## 2.1、查看系统是否已安装rsync

- 执行如下命令，看是否有返回结果

```bash
rpm -qa | grep rsync

# 返回结果如下，证明系统已安装rsync服务
rsync-3.1.2-10.el7.x86_64
```

- 如果没有安装的话，使用yum安装rsync服务即可

```bash
yum install -y rsync
```

## 2.2、配置rsyncd.conf文件

`rsyncd.conf` 文件默认是不存在，需要我们自己手动创建

1. 创建 `rsync` 配置文件的存放目录

```bash
mkdir /etc/rsyncd 
```

2. 编辑 `rsyncd.conf` 文件

```bash
cat > /etc/rsyncd/rsyncd.conf <<EOF
uid=0
gid=0
max connections=36000

log file=/var/log/rsyncd.log
pid file=/var/run/rsyncd.pid
lock file=/var/log/rsyncd.lock

#moth file=/etc/rsyncd/rsyncd.motd
read only = no
#hosts allow=10.0.0.0/8
#hosts deny=*
incoming chmod = D0755,F0644

[rsync]
    path=/mnt/Personal_Data/
    list=yes
    ignore errors
    auth users = rsync
    secrets file = /etc/rsyncd/rsyncd.secrets
EOF
```

> 配置文件说明：
>
> - `uid` ：设置 rsync 运行权限的用户的 uid；
> - `git` ：设置 rsync 运行权限的用户的 gid；
> - `max connections` ：设置最大连接数；
> - `log file` ：日志文件位置，启动 rsync 服务后启动创建该文件；
> - `pid file` ：pid文件的存放位置；
> - `lock file` ：锁文件的存放位置；
> - `read only` ：设置 rsync 服务端为读写权限；
> - `incoming chmod` ：设置文件和目录的权限，D代表目录，F代表文件；
> - `[rsync]` ：自定义的同步模块名称；
> - `path` ：rsync 服务端数据存放路径，客户端同步过来的数据存放在此目录；
> - `list` ：是否显示 rsync 服务端资源列表；
> - `ingore errors` ：如果出现错误，则忽略；
> - `auth users` ：rsync 进行数据同步的用户名，虚拟用户，不用在操作系统中创建；可以设置多个，用英文逗号隔开；
> - `secrets file` ：用户认证配置文件，里边保存用户名称和密码，需要手动创建；

3. 创建认证配置文件

```bash
echo "rsync:rsync" > /etc/rsyncd/rsyncd.secrets
```

4. 配置权限

```bash
chmod 600 /etc/rsyncd/rsyncd*

# 查看权限是否设置成功
ls -l /etc/rsyncd/
-rw------- 1 root root 384 May 24 18:21 rsyncd.conf
-rw------- 1 root root  12 May 23 22:54 rsyncd.secrets
```

## 2.3、启动rsync服务

```bash
rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```

2.4、查看rsync服务是否启动

- 方式一：使用 `lsof` 命令查看

```bash
[root@localhost ~]# lsof -i:873
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
rsync   49383 root    3u  IPv4 38923757      0t0  TCP *:rsync (LISTEN)
rsync   49383 root    5u  IPv6 38923758      0t0  TCP *:rsync (LISTEN)
```

- 方式二：使用 `ps` 命令查看

```bash
[root@localhost ~]# ps -ef | grep rsync
root     49383     1  0 18:21 ?        00:00:00 /usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
root     50210 49383  0 18:25 ?        00:00:01 /usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```

## 2.4、设置开机自启动

1. 查看 rsync 二进制文件的路径

```bash
[root@localhost ~]# which rsync
/usr/bin/rsync
```

2. 写入到/etc/rc.local文件中

```bash
echo "/usr/bin/rsync --daemon --config=/etc/rsyncd/rsyncd.conf" >> /etc/rc.local
```

# 三、ubuntu子系统配置rsync

## 3.1、查看系统是否支持inotify

```bash
root@PC-004785:/opt# ls -l /proc/sys/fs/inotify/
# 如果返回如下三个结果，说明操作系统是支持inotify的
total 0
-rw-r--r-- 1 root root 0 May 23 22:34 max_queued_events
-rw-r--r-- 1 root root 0 May 23 22:34 max_user_instances
-rw-r--r-- 1 root root 0 May 23 22:34 max_user_watches
```

## 3.2、安装inotify-tools

```bash
apt-get install inotify-tools -y
```

## 3.3、检查系统是否安装 rsync 服务

一般情况下， ubuntu 操作系统是安装好 rsync 服务的

```bash
apt list --installed | grep rsync    # 检查是否安装

# 返回类似下面的结果，表示已安装 rsync 服务
rsync/now 3.2.3-8ubuntu3.1 amd64 [installed,upgradable to: 3.2.7-0ubuntu0.22.04.2]
```

如果没有安装，使用以下命令安装 rsync 服务即可

```bash
apt-get install rsync -y
```

## 3.4、创建rsync同步密码文件

```bash
# 创建 rsync 配置文件目录
mkdir /etc/rsyncd

# 创建密码文件
echo "rsync" >> /etc/rsyncd/rsyncd.secrets
```

## 3.5、编写inotify监控同步脚本

- `vim /etc/rsyncd.inotify.sh`

```bash
host=10.202.43.110     # 目标服务器的ip(备份服务器)
src=/mnt/d/Personal_data/        # 在源服务器上所要监控的备份目录（此处可以自定义，但是要保证存在）
des=rsync     # 自定义的模块名，需要与目标服务器上定义的同步名称一致
password=/etc/rsyncd/rsyncd.secrets        # 执行数据同步的密码文件
user=rsync          # 执行数据同步的用户名
inotifywait=/usr/bin/inotifywait    # inotify监控工具的可执行路径，通过 which inotifywait 查看

$inotifywait -mrq --timefmt '%Y%m%d %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src \
| while read files;do
    rsync -avz --chmod=Du+rwx,g+rx,o+rx,Fa-x --delete  --timeout=100 --password-file=${password} $src $user@$host::$des
    echo "${files} was rsynced" >>/tmp/rsync.log 2>&1
done
```

## 3.6、后台启动脚本

后台启动：是为了让脚本实时监控要同步的目录下的所有文件，如果发生变化，则触发 rsync 同步功能

- 启动命令

```bash
nohup /bin/bash inotify.sh &
```



