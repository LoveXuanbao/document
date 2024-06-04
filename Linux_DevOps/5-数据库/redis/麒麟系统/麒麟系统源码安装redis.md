# redis官网源码包下载地址

```bash
https://download.redis.io/releases/?_gl=1*13t5s90*_ga*MTg3Mjk4MjQ4NC4xNjgzMzYyMjA2*_ga_8BKGRQKRPV*MTY4MzM2MjIwNi4xLjEuMTY4MzM2MjIwOS41Ny4wLjA.
```



# 安装

## 一、创建redis源码包存放目录

```bash
[root@localhost ~]# mkdir /tmp/redis
[root@localhost ~]# cd /tmp/redis
root@localhost redis]# ll
总用量 2240
-rw-r--r-- 1 root root 2247528  9月 10  2020 redis-6.0.8.tar.gz
```

## 二、解压源码包

```bash
[root@localhost redis]# tar -zxf redis-6.0.8.tar.gz 
[root@localhost redis]# ll
总用量 2240
drwxrwxr-x 7 root root     460  9月 10  2020 redis-6.0.8
-rw-r--r-- 1 root root 2247528  9月 10  2020 redis-6.0.8.tar.gz
[root@localhost redis]# cd redis-6.0.8
[root@localhost redis-6.0.8]# ll
总用量 1088
-rw-rw-r--  1 root root 96706  9月 10  2020 00-RELEASENOTES
-rw-rw-r--  1 root root    51  9月 10  2020 BUGS
-rw-rw-r--  1 root root  2381  9月 10  2020 CONTRIBUTING
-rw-rw-r--  1 root root  1487  9月 10  2020 COPYING
drwxrwxr-x  6 root root   180  9月 10  2020 deps
-rw-rw-r--  1 root root    11  9月 10  2020 INSTALL
-rw-rw-r--  1 root root   151  9月 10  2020 Makefile
-rw-rw-r--  1 root root  6888  9月 10  2020 MANIFESTO
-rw-rw-r--  1 root root 20987  9月 10  2020 README.md
-rw-rw-r--  1 root root 84642  9月 10  2020 redis.conf
-rwxrwxr-x  1 root root   275  9月 10  2020 runtest
-rwxrwxr-x  1 root root   280  9月 10  2020 runtest-cluster
-rwxrwxr-x  1 root root   761  9月 10  2020 runtest-moduleapi
-rwxrwxr-x  1 root root   281  9月 10  2020 runtest-sentinel
-rw-rw-r--  1 root root 10743  9月 10  2020 sentinel.conf
drwxrwxr-x  3 root root  2920  9月 10  2020 src
drwxrwxr-x 11 root root   260  9月 10  2020 tests
-rw-rw-r--  1 root root  3055  9月 10  2020 TLS.md
drwxrwxr-x  9 root root   480  9月 10  2020 utils
```

## 三、安装依赖

```bash
[root@localhost redis-6.0.8]# yum install -y gcc automake autoconf libtool make which libatomic
```

## 四、编译安装， 返回如下图所示表示编辑安装成功

```bash
[root@localhost redis-6.0.8]# make PREFIX=/usr/local/redis install
```

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230506173931.png)

- 查看/usr/local/redis/

```bash
# 编译后，在/usr/local/redis/目录下会生成一个 bin目录，存放的就是编译好的二进制启动文件
[root@localhost redis-6.0.8]# ll /usr/local/redis/bin/
总用量 36684
-rwxr-xr-x 1 root root 4923920  5月  6 17:08 redis-benchmark
-rwxr-xr-x 1 root root 9197136  5月  6 17:08 redis-check-aof
-rwxr-xr-x 1 root root 9197136  5月  6 17:08 redis-check-rdb
-rwxr-xr-x 1 root root 5037800  5月  6 17:08 redis-cli
lrwxrwxrwx 1 root root      12  5月  6 17:08 redis-sentinel -> redis-server
-rwxr-xr-x 1 root root 9197136  5月  6 17:08 redis-server
```

## 五、创建redis配置文件存放目录

```bash
[root@localhost redis-6.0.8]# mkdir /etc/redis
```

## 六、拷贝源码包中的redis.conf 到redis配置文件目录

```bash
[root@localhost redis-6.0.8]# cp redis.conf /etc/redis/
```

## 七、修改redis.conf文件

```bash
[root@localhost redis-6.0.8]# sed -i 's/bind 127.0.0.1/bind 0.0.0.0/' /etc/redis/redis.conf     # 允许所有主机连接
[root@localhost redis-6.0.8]# sed -i 's/daemonize no/daemonize yes/' /etc/redis/redis.conf      # 允许服务后台启动
[root@localhost redis-6.0.8]# sed -i 's/^# requirepass foobared/requirepass gridsum@2023!/' /etc/redis/redis.conf   # 修改密码
```

## 八、创建redis.service 

```bash
[root@localhost redis-6.0.8]# vim /usr/lib/systemd/system/redis.service
[Unit]
Description=Redis
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /etc/redis/redis.conf
ExecReload=/usr/local/redis/bin/redis-server -s reload
ExecStop=/usr/local/redis/bin/redis-server -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## 九、创建redis文件

```bash
[root@localhost redis-6.0.8]# vim /etc/init.d/redis
#!/bin/bash
### BEGIN INIT INFO
# Provides:                Redis
# Required-Start:
# Required-Stop:
# Default-Start:           2 3 4 5
# Default-Stop:            0 1 6
# Short-Description:       Start Redis daemon at boot time
# Description:             Start Redis daemon at boot time
### END INIT INFO

#redis服务器监听的端口
REDISPORT=6379
 
#服务端所处位置
EXEC=/usr/local/redis/bin/redis-server
 
#客户端位置
CLIEXEC=/usr/local/redis/bin/redis-cli
 
#redis的PID文件位置，按需修改
PIDFILE=/var/run/redis_${REDISPORT}.pid
 
#redis的配置文件位置
CONF="/etc/redis/redis.conf"
 
case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF &
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

## 十、赋予redis文件权限

```bash
[root@localhost redis-6.0.8]# chmod +x /etc/init.d/redis
```

## 十一、服务启动

```bash
[root@localhost redis-6.0.8]# systemctl start redis     # 启动redis服务
[root@localhost redis-6.0.8]# systemctl status redis    # 查看redis服务状态
● redis.service - Redis
   Loaded: loaded (/usr/lib/systemd/system/redis.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2023-05-06 17:29:07 CST; 18min ago
  Process: 30649 ExecStart=/usr/local/redis/bin/redis-server /etc/redis/redis.conf (code=exited, status=0/SUCCESS)
 Main PID: 30650 (redis-server)
    Tasks: 5
   Memory: 5.5M
   CGroup: /system.slice/redis.service
           └─30650 /usr/local/redis/bin/redis-server 0.0.0.0:6379

5月 06 17:29:07 localhost.localdomain systemd[1]: Starting Redis...
5月 06 17:29:07 localhost.localdomain systemd[1]: Started Redis.
```



# 客户端连接测试

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230506175444.png)



# 编译失败后的清理

```bash
[root@localhost redis-6.0.8]# make clean    # 清理
# 返回结果
cd src && make clean
make[1]: 进入目录“/tmp/redis/redis-6.0.8/src”
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
rm -f adlist.d quicklist.d ae.d anet.d dict.d server.d sds.d zmalloc.d lzf_c.d lzf_d.d pqsort.d zipmap.d sha1.d ziplist.d release.d networking.d util.d object.d db.d replication.d rdb.d t_string.d t_list.d t_set.d t_zset.d t_hash.d config.d aof.d pubsub.d multi.d debug.d sort.d intset.d syncio.d cluster.d crc16.d endianconv.d slowlog.d scripting.d bio.d rio.d rand.d memtest.d crcspeed.d crc64.d bitops.d sentinel.d notify.d setproctitle.d blocked.d hyperloglog.d latency.d sparkline.d redis-check-rdb.d redis-check-aof.d geo.d lazyfree.d module.d evict.d expire.d geohash.d geohash_helper.d childinfo.d defrag.d siphash.d rax.d t_stream.d listpack.d localtime.d lolwut.d lolwut5.d lolwut6.d acl.d gopher.d tracking.d connection.d tls.d sha256.d timeout.d setcpuaffinity.d anet.d adlist.d dict.d redis-cli.d zmalloc.d release.d ae.d crcspeed.d crc64.d siphash.d crc16.d ae.d anet.d redis-benchmark.d adlist.d dict.d zmalloc.d siphash.d
make[1]: 离开目录“/tmp/redis/redis-6.0.8/src”
[root@localhost redis-6.0.8]# mkdir distclean   # 清理
```

