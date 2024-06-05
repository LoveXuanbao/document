## docker服务启动异常报错

```bash
[root@gs-server-13406 docker]# systemctl restart docker
A dependency job for docker.service failed. See 'journalctl -xe' for details.
```
- 按照提示执行**journalctl -xe**查看报错

```bash
[root@gs-server-13406 docker]# journalctl -xe
...
-- Unit containerd.service has begun starting up.
Feb 08 03:10:39 gs-server-13406 systemd[17388]: Failed at step LIMITS spawning /sbin/modprobe: Operation not permitted
-- Subject: Process /sbin/modprobe could not be executed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- The process /sbin/modprobe could not be executed and failed.
-- 
-- The error number returned by this process is 1.
Feb 08 03:10:39 gs-server-13406 systemd[17392]: Failed at step LIMITS spawning /usr/bin/containerd: Operation not permitted
-- Subject: Process /usr/bin/containerd could not be executed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- The process /usr/bin/containerd could not be executed and failed.
-- 
-- The error number returned by this process is 1.
Feb 08 03:10:39 gs-server-13406 systemd[1]: containerd.service: main process exited, code=exited, status=205/LIMITS
Feb 08 03:10:39 gs-server-13406 systemd[1]: Failed to start containerd container runtime.
-- Subject: Unit containerd.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
-- 
-- Unit containerd.service has failed.
-- 
-- The result is failed.
```

- 查看系统日志信息

```bash
[root@gs-server-13406 docker]# tail -100f /var/log/messages
Feb  8 03:12:29 gs-server-13406 systemd: Starting containerd container runtime...
Feb  8 03:12:29 gs-server-13406 systemd: Failed at step LIMITS spawning /sbin/modprobe: Operation not permitted
Feb  8 03:12:29 gs-server-13406 systemd: Failed at step LIMITS spawning /usr/bin/containerd: Operation not permitted
Feb  8 03:12:29 gs-server-13406 systemd: containerd.service: main process exited, code=exited, status=205/LIMITS
Feb  8 03:12:29 gs-server-13406 systemd: Failed to start containerd container runtime.
Feb  8 03:12:29 gs-server-13406 systemd: Unit containerd.service entered failed state.
Feb  8 03:12:29 gs-server-13406 systemd: containerd.service failed.
Feb  8 03:12:33 gs-server-13406 systemd: Reloading.
Feb  8 03:12:39 gs-server-13406 systemd: containerd.service holdoff time over, scheduling restart.
Feb  8 03:12:39 gs-server-13406 systemd: Stopped containerd container runtime.
Feb  8 03:12:39 gs-server-13406 systemd: Starting containerd container runtime...
Feb  8 03:12:39 gs-server-13406 systemd: Failed at step LIMITS spawning /sbin/modprobe: Operation not permitted
Feb  8 03:12:39 gs-server-13406 systemd: Failed at step LIMITS spawning /usr/bin/containerd: Operation not permitted
Feb  8 03:12:39 gs-server-13406 systemd: containerd.service: main process exited, code=exited, status=205/LIMITS
Feb  8 03:12:39 gs-server-13406 systemd: Failed to start containerd container runtime.
Feb  8 03:12:39 gs-server-13406 systemd: Dependency failed for Docker Application Container Engine.
Feb  8 03:12:39 gs-server-13406 systemd: Job docker.service/start failed with result 'dependency'.
Feb  8 03:12:39 gs-server-13406 systemd: Unit containerd.service entered failed state.
Feb  8 03:12:39 gs-server-13406 systemd: containerd.service failed.
Feb  8 03:12:44 gs-server-13406 systemd: containerd.service holdoff time over, scheduling restart.
Feb  8 03:12:44 gs-server-13406 systemd: Stopped containerd container runtime.
Feb  8 03:12:44 gs-server-13406 systemd: Starting containerd container runtime...
Feb  8 03:12:44 gs-server-13406 systemd: Failed at step LIMITS spawning /sbin/modprobe: Operation not permitted
Feb  8 03:12:44 gs-server-13406 systemd: Failed at step LIMITS spawning /usr/bin/containerd: Operation not permitted
Feb  8 03:12:44 gs-server-13406 systemd: containerd.service: main process exited, code=exited, status=205/LIMITS
```

- 发现两个日志信息中关键错误信息为两个 **LIMITS**
- 查看系统limits

```bash
[root@gs-server-13406 docker]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63415
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 983040
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 63415
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

[root@gs-server-13406 docker]# lsof -Ki | wc -l   # 查看系统已经打开的文件数
3247
```

- 发现打开的文件数也没有超过系统设置的上限

- 查看docker和containerd的启动文件

```bash
cat /usr/lib/systemd/system/docker.service
...
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
...

cat /usr/lib/systemd/system/containerd.service
...
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=1048576
...
```

- 原因找到，containerd.service中配的**LimitNOFILE**超出系统可以打开的最大文件数



## 解决办法

- 修改docker.service和containerd.service中的所有**limit**相关配置

```bash
vim /usr/lib/systemd/system/docker.service
...
LimitNOFILE=65535
LimitNPROC=65535
LimitCORE=65535
...
vim /usr/lib/systemd/system/containerd.service
...
LimitNPROC=65535
LimitCORE=65535
LimitNOFILE=65535
...

```
