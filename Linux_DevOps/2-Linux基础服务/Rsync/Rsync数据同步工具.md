# 一、Rsync数据同步工具

## 1.1、什么是Rsync

- Remote synchronize远程同步，是一款开源的数据镜像备份工具，支持本地复制，或者与其他SSH、rsync主机同步；可以提供快速增量文件传输。
- 支持跨平台，可以在winodws与Linux之间进行数据同步。

## 1.2、Rsync的特性

1. **镜像：**可以镜像保存整个目录树和文件系统。支持匿名传输。
2. **属性不变：**可以很容器的做到保持原文件的权限、时间、符号链接等。

3. **快速：**第一次同步时，复制全部内容；第二次后只复制增加的部分；Rsync传输的时候可以实现压缩及解压缩操作，因此可以使用更少的带宽。

4. **安全：**可以使用scp、ssh等方式来传输文件；也可以通过socket链接。

## 1.3、Rsync的同步模式

### 1.3.1、拉复制

- 传输小文件没问题，传输大文件容易造成数据不一致
- 服务器端定义共享什么目录、共享资源名、认证用户、认证密码等
- 客户端定义间隔多长时间拉取服务器指定目录得数据，如果间隔时间为毫秒级时，同步小文件没有问题，但同步大文件时可能会造成数据不一致，且两端系统资源得消耗可能会比较大

### 1.3.2、推复制

推复制特点：传输大文件没问题，传输小文件时比较慢

- 监测到指定目录数据变化，即开始同步给远程主机

## 1.4、Rsync的三种工作模式

1. 本地模式

2. 通道模式

3. 守护进程模式

# 二、Rsync数据同步案例

## 2.1 服务端配置

### 2.1.1 安装rsync服务

```bash
yum install rsync

# 创建配置认证文件目录
mkdir /etc/rsyncd
```

### 2.1.2 添加rsync用户

```bash
useradd rsync -s /sbin/nologin -M
```

### 2.1.3 创建同步目录

```bash
mkdir /mnt/rsync

# 同步目录修改所有者权限
chown rsync.rsync -R /mnt/rsync
```

### 2.1.4 修改配置文件

```bash
# 备份原文件
cp /etc/rsyncd.conf /etc/rsyncd.conf.bak

#  vim 编辑配置文件
vim /etc/rsyncd.conf
uid=root
gid=root
# rsync端口
port=873
# 最大连接数
max connections=100
# 日志文件
log file=/var/log/rsyncd.log
# 进程号对应的pid文件
pod file=/var/run/rsyncd.pid
# 锁文件，防止文件不一致
lock file=/var/run/rsyncd.lock
# 指定一个消息文件，当客户端连接服务器时，将该文件内容显示给客户端
motd file=/etc/rsyncd/rsyncd.motd
# yes为只读，no为可读可写
read only=no
# 允许的网段
hosts allow=10.202.43.46/24
# 拒绝的网段
hosts deny=*
# 超时时间
timeout = 300

[www]
# 服务端提供访问的目录
path=/mnt/rsync
# 可以查看列表
list=yes
# 忽略错误
ignore errors
# 连接的虚拟用户，非系统用户
auth users=test
# 虚拟用户的密码文件
secrets file=/etc/rsyncd/rsyncd.secrets
# 记录传输文件的日志
transfer logging=yes
# 日志格式
log format=%t %a %m %f %b
# 日志级别
syslog facility=local3
```

### 2.1.5 创建消息文件(可选)

```bash
vim /etc/rsyncd/rsyncd.motd

####################
##welcome to rsync##
####################
```

### 2.1.6 创建密码文件

```bash
# 配置密码文件，格式为： 用户名:密码
cat > /etc/rsyncd/rsyncd.secrets <<EOF
test:123456   
EOF

# 修改文件权限，否则客户端无法完成验证
chmod 600 /etc/rsyncd/rsyncd.secrets
```

### 2.1.7 启动rsync服务

```bash
rsync --daemon --config=/etc/rsyncd.conf
```

### 2.1.8 验证服务是否启动

```bash
# 使用ps 查看
ps -ef | grep rsync 
root      2784 27495  0 15:35 pts/0    00:00:00 grep --color=auto rsync
root     60755     1  0 15:17 ?        00:00:00 rsync --daemon --config=/etc/rsyncd.conf   # 看是否有该进程


# 使用lsof 查看
lsof -i:873
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
rsync   60755 root    3u  IPv4 17765964      0t0  TCP *:rsync (LISTEN)
rsync   60755 root    5u  IPv6 17765965      0t0  TCP *:rsync (LISTEN)
```

## 2.2 客户端配置

### 2.2.1 配置认证文件

```bash
# 创建认证文件目录
mkdir /etc/rsyncd

# 配置认证文件，只填写密码
cat > /etc/rsyncd/rsyncd.secrets
123456 
EOF

# 修改权限
chmod 600 /etc/rsyncd/rsyncd.secrets
```

### 2.2.2 创建同步目录

```bash
mkdir /mnt/rsync
```

## 2.3 向服务端推送数据

```bash
[root@localhost rsync]# cd /mnt/rsync
[root@localhost rsync]# touch aaa.txt


# 命令格式: rsync [选项] 本地目录 用户名@server_ip::模块/
# 需要输入密码
[root@localhost rsync]# rsync -avzP /mnt/rsync test@10.202.43.110::www/  
###########
##welcome##
###########

Password: 
sending incremental file list
rsync/
rsync/aaa.txt
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=1/3)

sent 112 bytes  received 46 bytes  63.20 bytes/sec
total size is 0  speedup is 0.00

# 不需要输入密码
[root@localhost rsync]# touch bbb.txt

[root@localhost rsync]# rsync -avzP /mnt/rsync/ test@10.202.43.110::www/ --password-file=/etc/rsyncd/rsyncd.secrets

###########
##welcome##
###########

sending incremental file list
./
bbb.txt
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=0/3)

sent 134 bytes  received 46 bytes  360.00 bytes/sec
total size is 0  speedup is 0.00
```

## 2.4 向服务端拉取数据

```bash
# 服务端创建测试文件
[root@localhost rsync]# touch test{1..10}.txt
[root@localhost rsync]# ls
aaa.txt  bbb.txt  test10.txt  test1.txt  test2.txt  test3.txt  test4.txt  test5.txt  test6.txt  test7.txt  test8.txt  test9.txt


# 客户端执行拉取操作
[root@localhost rsync]# rsync -avzP --delete  test@10.202.43.110::www/ --password-file=/etc/rsyncd/rsyncd.secrets /mnt/rsync/
###########
##welcome##
###########

receiving incremental file list
test1.txt
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=9/13)
test10.txt
              0 100%    0.00kB/s    0:00:00 (xfr#2, to-chk=8/13)
test2.txt
              0 100%    0.00kB/s    0:00:00 (xfr#3, to-chk=7/13)
test3.txt
              0 100%    0.00kB/s    0:00:00 (xfr#4, to-chk=6/13)
test4.txt
              0 100%    0.00kB/s    0:00:00 (xfr#5, to-chk=5/13)
test5.txt
              0 100%    0.00kB/s    0:00:00 (xfr#6, to-chk=4/13)
test6.txt
              0 100%    0.00kB/s    0:00:00 (xfr#7, to-chk=3/13)
test7.txt
              0 100%    0.00kB/s    0:00:00 (xfr#8, to-chk=2/13)
test8.txt
              0 100%    0.00kB/s    0:00:00 (xfr#9, to-chk=1/13)
test9.txt
              0 100%    0.00kB/s    0:00:00 (xfr#10, to-chk=0/13)

sent 218 bytes  received 637 bytes  1,710.00 bytes/sec
total size is 0  speedup is 0.00
[root@localhost rsync]# ls
aaa.txt  bbb.txt  test10.txt  test1.txt  test2.txt  test3.txt  test4.txt  test5.txt  test6.txt  test7.txt  test8.txt  test9.txt
```

## 2.5 服务端向客户端推送数据

- 服务端监测到指定目录数据变化，即同步数据给远程主机

### 2.5.1 监测脚本下载地址

```bash
https://github.com/wsgzao/sersync/blob/master/sersync2.5.4_64bit_binary_stable_final.tar.gz
```

### 2.5.2 配置

#### 2.5.2.1 上传包到server主机

#### 2.5.2.2 解压监测工具

```bash
[root@localhost mnt]# tar -xf sersync2.5.4_64bit_binary_stable_final.tar.gz
[root@localhost mnt]# cd GNU-Linux-x86
[root@localhost GNU-Linux-x86]# ll
total 1772
-rwxr-xr-x 1 root root    2214 Oct 26  2011 confxml.xml
-rwxr-xr-x 1 root root 1810128 Oct 26  2011 sersync2
```

