官方文档：https://docs.starrocks.io/zh/docs/deployment/deployment_overview/

### 一、部署依赖

1. 操作系统：支持CentOS Linux 7.9 和 Ubuntu Linux 22.04
2. CPU支持AVX2指令集，AVX2指令集可以充分发挥其矢量化能力。可以用一下命令查看：

```
lscpu | grep avx2
```

3. 网络：建议使用万兆网络连接，确保StarRocks集群内数据能够跨节点高效传输；
4. JDK8；注意：StarRocks不支持JRE，如果需要在 Ubuntu 22.04 上部署，则必须安装 JDK 11

### 二、部署包准备

官方下载地址：

- starrocks: `https://www.starrocks.io/download/community`
- JDK: `https://www.oracle.com/java/technologies/downloads/#java8`

如果服务器能联网可以通过以下命令直接下载，或者使用本文档对应的离线部署包

```bash
# StarRocks:
wget https://releases.starrocks.io/starrocks/StarRocks-3.2.0-rc01.tar.gz

# JDK:
wget --no-check-certificate --no-cookies \
    --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
    http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
```



### 三、系统初始化

- CPUScaling Governor：用于控制CPU能耗模式；如果CPU支持该配置项，建议将其设置为`performance`以获得更好得CPU性能（耗电量较高，生产环境建议开启），如果不支持，则可以跳过该项（虚拟机大部分不支持）

```bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

- 备份原sysctl.conf文件，或者追加以下内容到 sysctl.conf文件中

```bash
# 允许操作系统将额外的内存资源分配给进程
vm.overcommit_memory = 1
# 禁用Transparent Huge Pages，系统默认启动，因其会干扰内存分配，进而导致性能下降，建议禁用
vm.nr_hugepages = 0
vm.hugetlb_shm_group = 0
# 关闭swappiness
vm.swappiness = 0
# 如果系统当前因后台进程无法处理的新连接而溢出，允许系统重置新连接
net.ipv4.tcp_abort_on_overflow = 1
# 设置监听 Socket 队列的最大连接请求数
net.core.somaxconn = 1024
```

- 如果集群负载并发较高，在 sysctl.conf 文件中追加以下配置

```bash
kernel.threads-max = 120000
vm.max_map_count = 262144
kernel.pid_max = 200000
```

- 执行以下命令是 sysctl.conf 中的配置生效

```bash
sysctl -p
```

- 禁用swap

```bash
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

- 禁用SELinux

```bash
sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sed -i 's/SELINUXTYPE/#SELINUXTYPE/' /etc/selinux/config
setenforce 0 
```

- 关闭防火墙

```bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

- 配置LANG变量

```bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

- 设置时区

```bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

- ulimit设置

```bash
# 最大文件打开数
echo "" >> /etc/security/limits.conf
echo "* soft nproc 65535" >> /etc/security/limits.conf
echo "* hard nproc 65535" >> /etc/security/limits.conf
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf
```

### 四、安装JDK

- 服务器创建java目录

```
mkdir -p /usr/local/java
```

- 解压jdk到指定目录

```bash
tar -zxf jdk-8u131-linux-x64.tar.gz -C /usr/local/java
```

- 添加 JAVA_HOME 到 /etc/profile

```bash
echo "" >> /etc/profile
echo 'export JAVA_HOME=/usr/local/java/jdk1.8.0_131' >> /etc/profile
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile
source /etc/profile
```

### 五、安装NTP

需要在 StarRocks 集群各个节点之间配置时间同步，从而保证事务的线性一致性。如果有现成的，用现成的即可，如果没有，按照下面方法安装

#### 5.1、在线安装

需要yum源能够直接安装ntp与ntpdate服务，然后执行下面命令

```bash
yum install -y ntp ntpdate 
```

#### 5.2、离线安装

离线安装包目录中有ntp目录，里面存在ntp的rpm包，执行下列命令安装即可

```bash
cd StarRocks/ntp
yum install -y ./*.rpm
```

#### 5.3、ntp配置

- server端，选择其中一台作为server节点，一般默认第一台

```bash
cp /etc/ntp.conf /etc/ntp.conf.bak
echo "" > /etc/ntp.conf

cat > /etc/ntp.conf <<EOF
driftfile /var/lib/ntp/drift
tinker panic 0

restrict default kod nomodify notrap
restrict 0.centos.pool.ntp.org nomodify notrap noquery
restrict 1.centos.pool.ntp.org nomodify notrap noquer
restrict 2.centos.pool.ntp.org nomodify notrap noquery
restrict 3.centos.pool.ntp.org nomodify notrap noquery
restrict 127.0.0.1
restrict ::1

server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
server 127.127.1.0 iburst
fudge 127.127.1.0 stratum 10

includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
EOF
```

- client端，注意修改server的ip为server端的IP

```bash
cp /etc/ntp.conf /etc/ntp.conf.bak
echo "" > /etc/ntp.conf

cat > /etc/ntp.conf <<EOF
driftfile /var/lib/ntp/drift
tinker panic 0

restrict default ignore
restrict 192.168.1.11 mask 255.255.255.255 nomodify notrap noquery
restrict 127.0.0.1
restrict ::1

server 192.168.1.11 iburst

includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
EOF
```

#### 5.4、启动NTP

```bash
systemctl enable ntpd --now    # 启动并添加开机自启动

# 常见操作：
systemctl stop ntpd   # 停止
systemctl start ntpd  # 启动
systemctl restart ntpd  # 重启
ntpstat   # 检查服务是否与NTP服务器server端同步
```

### 六、部署StarRocks

#### 6.1、解压StarRocks二进制部署包

使用以下命令解压二进制部署包到 /data目录；注意：/data/选择为实际环境数据盘安装

```bash
tar -zxf StarRocks-3.2.3.tar.gz -C /data1/
```

#### 6.2、创建数据存储路径

使用以下命令创建数据存储路径；注意：/data修改为实际环境的数据盘

```bash
mkdir /data1/starrocks-metadata   # fe元数据存储
```

#### 6.3、启动Leader FE节点

- 备份初始配置文件

```bash
cp /data1/StarRocks-3.2.3/fe/conf/fe.conf /data1/StarRocks-3.2.3/fe/conf/fe.conf.bak
```

- 编辑 fe.conf 文件，把原内容清空，把以下内容粘贴进去，，注意修改：`frontend_address` 和 `priority_networks`, 其他配置按需修改，然后保存

```bash
vim /data1/StarRocks-3.2.3/fe/conf/fe.conf
```

```bash
DATE = "$(date +%Y%m%d-%H%M%S)"
JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx8192m -XX:+UseMembar -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=7 -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:-CMSParallelRemarkEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:SoftRefLRUPolicyMSPerMB=0 -Xloggc:${LOG_DIR}/fe.gc.log.$DATE -XX:+PrintConcurrentLocks -Djava.security.policy=${STARROCKS_HOME}/conf/udf_security.policy"
JAVA_OPTS_FOR_JDK_11="-Dlog4j2.formatMsgNoLookups=true -Xmx8192m -XX:+UseG1GC -Xlog:gc*:${LOG_DIR}/fe.gc.log.$DATE:time -Djava.security.policy=${STARROCKS_HOME}/conf/udf_security.policy"

http_port = 8030
rpc_port = 9020
query_port = 9030
edit_log_port = 9010

# FE 所在 StarRocks 集群的 ID。具有相同集群 ID 的 FE 或 BE 属于同一个 StarRocks 集群
cluster_id = 100
# 元数据保存目录
meta_dir = /data/starrocks-metadata

# 日志文件路径
LOG_DIR = /data/StarRocks-3.2.0-rc01/fe/log
# 日志文件的大小
log_roll_size_mb = 1024
# 日志文件的保留时长 
sys_log_delete_age = 10d
# 审计日志的保留时长
audit_log_delete_age = 30d
# Dump日志的保留时长
dump_log_delete_age = 7d

# FE节点的IP地址
# 修改为当前部署节点IP
frontend_address = 192.168.1.11
# 修改为当前部署节点IP的CIDR
priority_networks = 192.168.1.0/24

# FE所在的 StarRocks 集群的名称，显示为网页标题
cluster_name = cluster_fe

# Thrift服务器支持的最大工作线程数
thrift_server_max_worker_threads
# Thrift 客户端链接的空闲超时时间，即链接超过该时间无新请求后则将链接断开
thrift_client_timeout_ms = 5000
# hrift 服务器 pending 队列长度。如果当前处理线程数量超过了配置项 thrift_server_max_worker_threads 的值，则将超出的线程加入 pending 队列。
thrift_server_queue_size = 4096
# bRPC 的空闲等待时间。单位：毫秒。
brpc_idle_wait_max_time = 10000

# 系统日志级别：INFO、WARN、ERROR、FATAL
sys_log_level = INFO

# 是否开启 MySQL 服务器的异步 I/O 选项
mysql_service_nio_enabled = true
# MySQL 服务器中用于处理 I/O 事件的最大线程数
mysql_service_io_threads_num = 4
# MySQL 服务器支持的 Backlog 队列长度
mysql_nio_backlog_num = 1024
# MySQL 服务器中用于处理任务的最大线程数
max_mysql_service_task_threads_num = 4096

# 连接调度器支持的最大线程数
max_connection_scheduler_threads_num = 4096

# FE 支持的最大连接数，包括所有用户发起的连接
qe_max_connection = 2048
```

- 启动FE节点

```bash
cd /data1/StarRocks-3.2.3/fe/bin/
./start_fe.sh --daemon
```

- 查看 FE 日志，检查 FE 节点是否启动成功

```bash
cat /data1/StarRocks-3.2.3/fe/log/fe.log | grep thrift
```

如果日志打印以下内容，则说明该 FE 节点启动成功

```bash
2024-03-19 11:03:05,908 INFO (UNKNOWN 192.168.1.11_9010_1710817376923(-1)|1) [FeServer.start():65] thrift server started with port 9020.
```

#### 6.4、启动高可用 FE 集群（可选）

高可用 FE 集群需要在 StarRocks 集群中部署至少三个 FE 节点，如果部署高可用 FE 集群，需要额外再启动两个新的 FE 节点，部署方式参考 6.3 章节，仅需要修改 fe.conf 文件中的 `frontend_address` 与启动方式即可

- 向集群中添加新的 Follower FE 节点时，必须在首次启动新 FE 节点时为其分配一个 helper节点（本质上是一个现有的 Follower FE节点），以同步所有 FE 元数据信息，注意：只需要在第一次启动其他节点的时候添加 --helper 参数，后续重启不需要添加

```bash
# 将 <helper_fe_ip> 替换为 Leader FE 节点的 IP 地址（priority_networks），
# 并将 <helper_edit_log_port>（默认：9010）替换为 Leader FE 节点的 edit_log_port。
/data/StarRocks-3.2.0-rc01/fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon

例如：
/data/StarRocks-3.2.0-rc01/fe/bin/start_fe.sh --helper 192.168.1.11:9010 --daemon
```

- 通过MySQL 客户端连接到 StarRocks，需要使用初始用户 root 登录，密码默认为空

```bash
# 将 <fe_address> 替换为 Leader FE 节点的 IP 地址（priority_networks）或 FQDN，
# 并将 <query_port>（默认：9030）替换为您在 fe.conf 中指定的 query_port。
mysql -h <fe_address> -P<query_port> -uroot

例如：
mysql -h 192.168.1.11 -P9030 -uroot
```

- 执行以下 SQL 将额外的 FE 节点添加至集群

一次只能通过一条 SQL 添加一个 Follower 节点

```sql
-- 将 <new_fe_address> 替换为您需要添加的新 FE 节点的 IP 地址（priority_networks）或 FQDN，
-- 并将 <edit_log_port>（默认：9010）替换为您在新 FE 节点的 fe.conf 中指定的 edit_log_port。
ALTER SYSTEM ADD FOLLOWER "<new_fe_address>:<edit_log_port>";

例如：
ALTER SYSTEM ADD FOLLOWER "192.168.1.12:9010";
ALTER SYSTEM ADD FOLLOWER "192.168.1.13:9010";
```

- 通过 MySQL 客户端查看 FE 节点状态

```bash
mysql> SHOW PROC '/frontends'\G
*************************** 1. row ***************************
             Name: 192.168.1.12_9010_1702895515663
               IP: 192.168.1.12
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: LEADER
        ClusterId: 100
             Join: true
            Alive: true
ReplayedJournalId: 3471
    LastHeartbeat: 2023-12-20 16:13:20
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-12-18 18:34:13
          Version: 3.2.0-rc01-9d64ad2
*************************** 2. row ***************************
             Name: 192.168.1.11_9010_1702893092966
               IP: 192.168.1.11
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: FOLLOWER
        ClusterId: 100
             Join: true
            Alive: true
ReplayedJournalId: 3472
    LastHeartbeat: 2023-12-20 16:13:20
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-12-20 14:47:03
          Version: 3.2.0-rc01-9d64ad2
*************************** 3. row ***************************
             Name: 192.168.1.13_9010_1702895518777
               IP: 192.168.1.13
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: FOLLOWER
        ClusterId: 100
             Join: true
            Alive: true
ReplayedJournalId: 3471
    LastHeartbeat: 2023-12-20 16:13:20
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-12-18 18:36:20
          Version: 3.2.0-rc01-9d64ad2
3 rows in set (0.06 sec)
```

#### 6.4、启动 BE 服务

所有 BE 节点重复下列步骤即可，无需其他操作

- 创建 BE 数据存储路径

```bash
mkdir /data1/starrocks-data       # be数据存储目录
```

- 备份初始配置文件

```bash
cp /data1/StarRocks-3.2.3/be/conf/be.conf /data1/StarRocks-3.2.3/be/conf/be.conf.bak
```

- 编辑 be.conf 文件，把原内容清空，把以下内容粘贴进去， 其他配置按需修改，然后保存

```bash
vim /data1/StarRocks-3.2.3/be/conf/be.conf
```

```bash
be_port = 9060
be_http_port = 8040
heartbeat_service_port = 9050
brpc_port = 8060

sys_log_dir = ${STARROCKS_HOME}/log
sys_log_level = INFO

# 日志拆分的大小，每 1G 拆分一个日志
sys_log_roll_mode = SIZE-MB-1024

# 日志保留的数目
sys_log_roll_num = 10

# 数据存储路径
storage_root_path = /data/starrocks-data

# UDF 程序存放的路径
user_function_dir = ${STARROCKS_HOME}/lib/udf

# 保存文件管理器下载的文件的目录
small_file_dir = ${STARROCKS_HOME}/lib/small_file

# 文件句柄缓存的容量
file_descriptor_cache_capacity = 16384

# 最小查询线程数，默认启动 64 个线程
fragment_pool_thread_num_min = 64

# 最大查询线程数
fragment_pool_thread_num_max = 4096

# 单节点上能够处理的查询请求上限
fragment_pool_queue_size = 2048

# BE 进程内存上限
mem_limit = 70%

# 磁盘路径。支持添加多个路径，多个路径之间使用分号(;) 隔开。建议 BE 机器有几个磁盘即添加几个路径。配置路径后，StarRocks 会自动创建名为 cachelib_data 的文件用于缓存 block
block_cache_disk_path = /data

# Block 的元数据存储目录
block_cache_meta_path = $STARROCKS_HOME

# 单个 block 大小，单位：字节
# block_cache_block_size =   1048576

# 内存缓存数据量的上限，单位：字节,生产环境推荐该参数数值最低设置成20GB，在开启 Data Cache 期间，存在大量从磁盘读取数据的情况，可考虑调大该参数
# block_cache_mem_size = 2147483648

# 单个磁盘缓存数据量的上限。举例：在 block_cache_disk_path 中配置了 2 个磁盘，并设置 block_cache_disk_size 参数值为 21474836480，即 20 GB，那么最多可缓存 40 GB 的磁盘数据
# block_cache_disk_size = 0
```

- 启动 BE 节点

```bash
cd /data1/StarRocks-3.2.3/be/bin
./start_be.sh --daemon
```

- 查看 BE 日志，检查 BE 节点是否启动成功

```bash
cat /data1/StarRocks-3.2.3/be/log/be.INFO | grep heartbeat
```

如果日志打印以下内容，则说明该 BE 节点启动成功

```bash
I1220 15:25:43.388882  2152 starrocks_be.cpp:258] BE start step 13: start heartbeat server successfully
I1220 15:25:43.787151  4234 thrift_server.cpp:377] heartbeat has started listening port on 9050
```

#### 6.5、组建集群

当所有的 FE、BE节点启动成功后，即可组建 StarRocks集群

- 找一台能与 FE 节点通信的主机，或者在 FE 节点上，安装 mysql-client；离线包里有

```bash
tar -zxvf mysql-client.tar.gz 
cd mysql-client
yum install -y ./*.rpm
```

- 通过 MySQL 客户端连接到 StarRocks。使用初始用户 root 登录，密码默认为空

```bash
# 将 <fe_address> 替换为 Leader FE 节点的 IP 地址（priority_networks）或 FQDN，
# 并将 <query_port>（默认：9030）替换为您在 fe.conf 中指定的 query_port。
mysql -h <fe_address> -P<query_port> -uroot

例如：
mysql -h 192.168.1.11 -P9030 -uroot
```

- 执行以下SQL查看 Leader FE 节点状态

```bash
SHOW PROC '/frontends'\G
```

```bash
mysql> SHOW PROC '/frontends'\G
*************************** 1. row ***************************
             Name: 192.168.1.12_9010_1702895515663
               IP: 192.168.1.12
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: LEADER   # 如果该字段为 LEADER，说明该 FE 节点为 Leader FE节点，如果为 FOLLOWER，说明该 FE 节点有资格被选为 Leader FE 节点
        ClusterId: 100
             Join: true
            Alive: true     # 如果该字段为true，说明该 FE 节点正常启动并机器如集群
ReplayedJournalId: 2911
    LastHeartbeat: 2023-12-20 15:42:10
         IsHelper: true
           ErrMsg: 
        StartTime: 2023-12-18 18:34:13
          Version: 3.2.0-rc01-9d64ad2
```

- 添加 BE 节点至集群

```SQL
-- 将 <be_address> 替换为 BE 节点的 IP 地址（priority_networks）或 FQDN，
-- 并将 <heartbeat_service_port>（默认：9050）替换为您在 be.conf 中指定的 heartbeat_service_port。
ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>", "<be2_address>:<heartbeat_service_port>", "<be3_address>:<heartbeat_service_port>";

例如添加三台 BE 节点：
ALTER SYSTEM ADD BACKEND "192.168.1.11:9050", "192.168.1.12:9050", "192.168.1.13:9050";
```

- 执行以下 SQL 查看 BE 节点状态

```SQL
SHOW PROC '/backends'\G
```

```bash
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
            BackendId: 10005
                   IP: 192.168.1.11
        HeartbeatPort: 9050
               BePort: 9060
             HttpPort: 8040
             BrpcPort: 8060
        LastStartTime: 2023-12-18 18:30:34
        LastHeartbeat: 2023-12-18 18:30:34
                Alive: true         # 如果该字段为true，则说明该 BE 节点正常启动并加入集群
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 48
     DataUsedCapacity: 0.000 B
        AvailCapacity: 7.922 GB
        TotalCapacity: 16.986 GB
              UsedPct: 53.36 %
       MaxDiskUsedPct: 53.36 %
               ErrMsg: 
              Version: 3.2.0-rc01-9d64ad2
               Status: {"lastSuccessReportTabletsTime":"N/A"}
    DataTotalCapacity: 7.922 GB
          DataUsedPct: 0.00 %
             CpuCores: 2
    NumRunningQueries: 0
           MemUsedPct: 0.00 %
           CpuUsedPct: 0.0 %
```

### 七、部署后设置

- 设置 MySQL 密码

  1. 使用用户名 `root` 和空密码通过 MySQL 客户端连接到 StarRocks

  ```bash
  # 将 <fe_address> 替换为您连接的 FE 节点的 IP 地址（priority_networks）
  # 或 FQDN，将 <query_port> 替换为您在 fe.conf 中指定的 query_port（默认：9030）。
  mysql -h <fe_address> -P<query_port> -uroot
  
  例如：
  mysql -h 192.168.1.11 -P9030 -uroot
  ```

  2. 执行以下 SQL 重置 `root` 用户密码

  ```bash
  -- 将 <password> 替换为您要为 root 用户设置的密码。
  SET PASSWORD = PASSWORD('<password>')
  
  例如：
  SET PASSWORD = PASSWORD('1qaz@WSX');
  ```

- 设置必要的系统变量

  ```sql
  -- 全局设置 is_report_success 为 false
  SET GLOBAL is_report_success = false;
  -- 全局设置 enable_profile 为 false
  SET GLOBAL enable_profile = false;
  -- 全局设置 enable_pipeline_engine 为 true
  SET GLOBAL enable_pipeline_engine = true;
  -- 全局设置 parallel_fragment_exec_instance_num 为 1
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  -- 全局设置 pipeline_dop 为 0
  SET GLOBAL pipeline_dop = 0;
  ```

### 八、StarRocks集群启动停止操作

- 停止 FE 节点

```bash
/data/StarRocks-3.2.0-rc01/fe/bin/stop_fe.sh --daemon
```

- 启动 FE 节点

```bash
/data/StarRocks-3.2.0-rc01/fe/bin/start_fe.sh --daemon
```

- 停止 BE 节点

```bash
/data/StarRocks-3.2.0-rc01/be/bin/stop_be.sh --daemon
```

- 启动 BE 节点

```bash
/data/StarRocks-3.2.0-rc01/be/bin/start_be.sh --daemon
```

