# 单节点部署

## 一、下载部署包

```bash
https://www.elastic.co/cn/downloads/past-releases#elasticsearch
```

选择服务，选择版本，点击Download

![image-20230825162524156](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162524156.png)

点击**LINUX X86_64** 开始下载，下载后得安装包名为 **elasticsearch-7.16.1-linux-x86_64.tar.gz**

![image-20230825162603604](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162603604.png)

## 二、上传部署包到服务器

上传 **elasticsearch-7.16.1-linux-x86_64.tar.gz**到服务器/tmp目录下

```bash
ll /tmp

# 返回结果
-rw-r--r--  1 root root 342821526 Mar 27 16:32 elasticsearch-7.16.1-linux-x86_64.tar.gz
```

## 三、创建es数据目录

在服务器磁盘空间重组得路径下，创建es-cluster目录

```bash
mkdir -p /data1/es-cluster
```

## 四、将安装包解压到es-cluster目录

```bash
tar -zxvf elasticsearch-7.16.1-linux-x86_64.tar.gz -C /data1/es-cluster/    # 解压

ll /data1/es-cluster/    # 查看es-cluster目录

# 返回结果
drwxr-xr-x 9 root root 155 Dec 11  2021 elasticsearch-7.16.1

ll /data1/es-cluster/elasticsearch-7.16.1/    # 查看elasitcsearch-7.16.1目录

# 返回结果
drwxr-xr-x  2 root root   4096 Dec 11  2021 bin
drwxr-xr-x  3 root root    210 Mar 27 16:39 config
drwxr-xr-x  9 root root    121 Dec 11  2021 jdk
drwxr-xr-x  3 root root   4096 Dec 11  2021 lib
-rw-r--r--  1 root root   3860 Dec 11  2021 LICENSE.txt
drwxr-xr-x  2 root root      6 Dec 11  2021 logs
drwxr-xr-x 61 root root   4096 Dec 11  2021 modules
-rw-r--r--  1 root root 627787 Dec 11  2021 NOTICE.txt
drwxr-xr-x  2 root root      6 Dec 11  2021 plugins
-rw-r--r--  1 root root   2710 Dec 11  2021 README.asciidoc
```

## 五、创建elasitc用户

```bash
useradd elastic    # 创建用户
echo '123456' | passwd --stdin elastic    # 通过echo密码给passwd --stdin修改
```

## 六、修改es-cluster目录权限

```bash
chown -R elastic.elastic /data1/es-cluster
```

## 七、配置文件修改

### 1、关闭SWAP交换内存

es官方网站建议关闭swap(https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html#_swapping_%E6%98%AF%E6%80%A7%E8%83%BD%E7%9A%84%E5%9D%9F%E5%A2%93)

```bash
swapoff -a 
    sed -i '/ swap / s/^/#/' /etc/fstab
```



### 2、修改limits参数

```bash
cat >> /etc/security/limits.conf <<EOF

*    soft    nofile    65536
*    hard    nofile    65536
*    soft    nproc     32000
*    hard    nproc     32000
*    hard    memlock   unlimited
*    soft    memlock   unlimited
EOF
```

### 3、修改sysctl.conf

```bash
vim /etc/sysctl.conf 

# ES内存映射文件系统，修改配置值得最大映射数量，以便又足够得虚拟内存用于mmapped文件
vm.max_map_count=655360

# 对上面得内容使其生效
sysctl -p
```

### 4、修改内存配置

- 建议修改为物理内存得一半大小，不要超过32G，参考文章(https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html)

```bash
vim /data1/es-cluster/elasticsearch-7.16.1/config/jvm.options
```

取消注释，并修改为合适得大小

![image-20230825162640295](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162640295.png)

### 5、修改es配置文件

- 备份源配置文件

```bash
cp /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml \
   /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml.bak
```

- 修改配置文件
  - 参数说明官方文档(https://www.elastic.co/guide/en/elasticsearch/reference/7.16/important-settings.html)

```bash
cat > /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml <<EOF
# 指定集群名字
cluster.name: es-cluster
# 指定节点名字
node.name: node-1
# 表示这个节点是否具有主节点的资格
# node.master: true
# 表示这个节点是否存储数据
node.data: true
# 数据存放目录
path.data: /data1/es-cluster/elasticsearch-7.16.1/data
# 日志存放目录
path.logs: /data1/es-cluster/elasticsearch-7.16.1/logs
# 内存锁检查
bootstrap.memory_lock: true
# http传输流量地址
network.host: 10.202.43.46
# 客户端通信绑定的端口
http.port: 9200
# 启动或禁用跨源资源共享
http.cors.enabled: true
# 允许哪些源请求开放，*代表所有
http.cors.allow-origin: "*"
# 节点之间的通信绑定的端口
transport.port: 9300
# 提供可访问的集群列表，没有给出端口的，通过transport.port确定端口
discovery.seed_hosts: ["10.202.43.46:9300"]
# 设置全新集群中符合主机资格的节点的初始集合（首次启动集群时需要），默认为空表示希望该节点加入已经被引导的集群
cluster.initial_master_nodes: ["node-1"]
# 启动es安全功能
xpack.security.enabled: true
# 用于节点通信是否启动或禁用TLS/SSL
xpack.security.transport.ssl.enabled: true
# 控制证书的验证
xpack.security.transport.ssl.verification_mode: certificate
# 包含私钥和证书的密钥文件路径
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
# 信任证书的密钥库路径
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
#集群内同时启动的数据任务个数，默认是2个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认4个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
# 适当增大搜索线程，提高搜索性能和稳定性
thread_pool.search.size: 24
thread_pool.search.queue_size: 500
# 适当增大写入buffer和队列长度，提高写入性能和稳定性
indices.memory.index_buffer_size: 15%
thread_pool.write.queue_size: 1024
# 防止新建shard时扫描所有shard的元数据，提升shard分配速度
cluster.routing.allocation.disk.include_relocations: false
EOF
```

### 6、生成用户名和密码

#### 1、生成证书

```bash
su - elastic
cd /data1/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""

# 返回结果
...
Certificates written to /data/es-cluster/elasticsearch-7.16.1/config/elastic-certificates.p12

This file should be properly secured as it contains the private key for 
your instance.

This file is a self contained file and can be copied and used 'as is'
For each Elastic product that you wish to configure, you should copy
this '.p12' file to the relevant configuration directory
and then follow the SSL configuration instructions in the product guide.
```

#### 2、手动生成密码

```bash
su - elastic
cd /data1/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:      # 输入指定的密码，此处在设置的时候不会显示
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana_system]: 
Reenter password for [kibana_system]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

#### 3、自动生成密码

```bash
su - elastic
cd /data/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-setup-passwords auto

# 如果出现以下错误，说明还没有启动ES服务，启动服务之后，重新执行生成密码
ERROR: Elasticsearch keystore file is missing [/data/es-cluster/elasticsearch-7.16.1/config/elasticsearch.keystore]

# 正常返回结果， 请记录以下密码
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y


Changed password for user apm_system
PASSWORD apm_system = t9J72TFapzfQqeEHZoz3

Changed password for user kibana_system
PASSWORD kibana_system = DpskmVpvdTA8qBm8KFXc

Changed password for user kibana
PASSWORD kibana = DpskmVpvdTA8qBm8KFXc

Changed password for user logstash_system
PASSWORD logstash_system = uN44HUAwgZPmGXw3tkM4

Changed password for user beats_system
PASSWORD beats_system = L8a4WHGDq3kmXYtGfSEH

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = GOFTcS2NTJMQJ29HZrfi

Changed password for user elastic
PASSWORD elastic = n1Uos6YOeBQYtn2UISWL
```

## 八、启动ES

```bash
su - elastic
cd /data/es-cluster/elasticsearch-7.16.1/bin
./elasticsearch -d
```

## 九、浏览器访问验证

浏览器输入IP:9200，然后使用elastic账号密码登录即可

![](https://typora-1306014148.cos.ap-nanjing.myqcloud.com/markdown/20230328175820.png)

# 集群部署

## 一、集群主机规划

| 服务器IP     | ES角色 | 操作系统   | CPU  | 内存 | 磁盘 |
| ------------ | ------ | ---------- | ---- | ---- | ---- |
| 10.202.43.46 | node-1 | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.55 | node-2 | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.56 | node-3 | CentOS 7.8 | 4C   | 8G   | 200G |

## 二、创建es数据目录

- 所有主机操作；在服务器磁盘空间重组得路径下，创建es-cluster目录

```bash
mkdir -p /data1/es-cluster
```

## 三、将安装包解压到es-cluster目录

- 所有主机操作

```bash
tar -zxvf elasticsearch-7.16.1-linux-x86_64.tar.gz -C /data1/es-cluster/    # 解压

ll /data1/es-cluster/    # 查看es-cluster目录

# 返回结果
drwxr-xr-x 9 root root 155 Dec 11  2021 elasticsearch-7.16.1

ll /data1/es-cluster/elasticsearch-7.16.1/    # 查看elasitcsearch-7.16.1目录

# 返回结果
drwxr-xr-x  2 root root   4096 Dec 11  2021 bin
drwxr-xr-x  3 root root    210 Mar 27 16:39 config
drwxr-xr-x  9 root root    121 Dec 11  2021 jdk
drwxr-xr-x  3 root root   4096 Dec 11  2021 lib
-rw-r--r--  1 root root   3860 Dec 11  2021 LICENSE.txt
drwxr-xr-x  2 root root      6 Dec 11  2021 logs
drwxr-xr-x 61 root root   4096 Dec 11  2021 modules
-rw-r--r--  1 root root 627787 Dec 11  2021 NOTICE.txt
drwxr-xr-x  2 root root      6 Dec 11  2021 plugins
-rw-r--r--  1 root root   2710 Dec 11  2021 README.asciidoc
```

## 四、创建elasitc用户

- 所有主机操作

```bash
useradd elastic    # 创建用户
echo 'elastic' | passwd --stdin elastic    # 通过echo密码给passwd --stdin修改
```

## 五、修改es-cluster目录权限

- 所有主机操作

```bash
chown -R elastic.elastic /data1/es-cluster
```

## 六、配置文件修改

### 1、关闭SWAP交换内存

- 所有主机操作

es官方网站建议关闭swap(https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html#_swapping_%E6%98%AF%E6%80%A7%E8%83%BD%E7%9A%84%E5%9D%9F%E5%A2%93)

```bash
swapoff -a 
sed -i '/ swap / s/^/#/' /etc/fstab
```

### 2、修改limits参数

- 所有主机操作

```bash
cat >> /etc/security/limits.conf <<EOF

*    soft    nofile    65536
*    hard    nofile    65536
*    soft    nproc     32000
*    hard    nproc     32000
*    hard    memlock   unlimited
*    soft    memlock   unlimited
EOF
```

### 3、修改sysctl.conf

- 所有主机操作

```bash
vim /etc/sysctl.conf 

# ES内存映射文件系统，修改配置值得最大映射数量，以便又足够得虚拟内存用于mmapped文件
vm.max_map_count=655360

# 对上面得内容使其生效
sysctl -p
```

### 4、修改内存配置

- 所有主机操作

- 建议修改为物理内存得一半大小，不要超过32G，参考文章(https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html)

```bash
vim /data1/es-cluster/elasticsearch-7.16.1/config/jvm.options
```

取消注释，并修改为合适得大小

![image-20230825162640295](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162640295.png)

### 5、修改es配置文件

- 备份源配置文件，所有主机操作

```bash
cp /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml \
   /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml.bak
```

- 修改配置文件
  - 参数说明官方文档(https://www.elastic.co/guide/en/elasticsearch/reference/7.16/important-settings.html)

```bash
cat > /data1/es-cluster/elasticsearch-7.16.1/config/elasticsearch.yml <<EOF
# 指定集群名字
cluster.name: es-cluster
# 指定节点名字，集群模式，不同主机要修改对应的角色名称,node-2/node-3等
node.name: node-1
# 表示这个节点是否具有主节点的资格
# node.master: true
# 表示这个节点是否存储数据
node.data: true
# 数据存放目录
path.data: /data1/es-cluster/elasticsearch-7.16.1/data
# 日志存放目录
path.logs: /data1/es-cluster/elasticsearch-7.16.1/logs
# 内存锁
bootstrap.memory_lock: true
# http传输流量地址，集群模式，不同的主机要修改为不同的主机IP
network.host: 10.202.43.46
# 客户端通信绑定的端口
http.port: 9200
# 启动或禁用跨源资源共享
http.cors.enabled: true
# 允许哪些源请求开放，*代表所有
http.cors.allow-origin: "*"
# 节点之间的通信绑定的端口
transport.port: 9300
# 提供可访问的集群列表，没有给出端口的，通过transport.port确定端口
discovery.seed_hosts: ["10.202.43.46:9300", "10.202.43.55:9300", "10.202.43.56:9300"]
# 设置全新集群中符合主机资格的节点的初始集合（首次启动集群时需要），默认为空表示希望该节点加入已经被引导的集群
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
# 启动es安全功能
xpack.security.enabled: true
# 用于节点通信是否启动或禁用TLS/SSL
xpack.security.transport.ssl.enabled: true
# 控制证书的验证
xpack.security.transport.ssl.verification_mode: certificate
# 包含私钥和证书的密钥文件路径
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
# 信任证书的密钥库路径
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
#集群内同时启动的数据任务个数，默认是2个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认4个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
# 适当增大搜索线程，提高搜索性能和稳定性
thread_pool.search.size: 24
thread_pool.search.queue_size: 500
# 适当增大写入buffer和队列长度，提高写入性能和稳定性
indices.memory.index_buffer_size: 15%
thread_pool.write.queue_size: 1024
# 防止新建shard时扫描所有shard的元数据，提升shard分配速度
cluster.routing.allocation.disk.include_relocations: false
EOF
```

### 6、生成用户名和密码

#### 1、生成证书

```bash
su - elastic
cd /data1/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""

# 返回结果
...
Certificates written to /data/es-cluster/elasticsearch-7.16.1/config/elastic-certificates.p12

This file should be properly secured as it contains the private key for 
your instance.

This file is a self contained file and can be copied and used 'as is'
For each Elastic product that you wish to configure, you should copy
this '.p12' file to the relevant configuration directory
and then follow the SSL configuration instructions in the product guide.
```

#### 2、手动生成密码

```bash
su - elastic
cd /data/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-setup-passwords interactive
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:      # 输入指定的密码，此处在设置的时候不会显示
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana_system]: 
Reenter password for [kibana_system]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

#### 3、自动生成密码

```bash
su - elastic
cd /data/es-cluster/elasticsearch-7.16.1/bin/
./elasticsearch-setup-passwords auto

# 如果出现以下错误，说明还没有启动ES服务，启动服务之后，重新执行生成密码
ERROR: Elasticsearch keystore file is missing [/data/es-cluster/elasticsearch-7.16.1/config/elasticsearch.keystore]

# 正常返回结果， 请记录以下密码
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y


Changed password for user apm_system
PASSWORD apm_system = t9J72TFapzfQqeEHZoz3

Changed password for user kibana_system
PASSWORD kibana_system = DpskmVpvdTA8qBm8KFXc

Changed password for user kibana
PASSWORD kibana = DpskmVpvdTA8qBm8KFXc

Changed password for user logstash_system
PASSWORD logstash_system = uN44HUAwgZPmGXw3tkM4

Changed password for user beats_system
PASSWORD beats_system = L8a4WHGDq3kmXYtGfSEH

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = GOFTcS2NTJMQJ29HZrfi

Changed password for user elastic
PASSWORD elastic = n1Uos6YOeBQYtn2UISWL
```

## 七、启动ES

```bash
su - elastic
cd /data1/es-cluster/elasticsearch-7.16.1/bin
./elasticsearch -d
```

## 八、浏览器访问验证

使用IP:9200/_cat/nodes，然后使用elastic账号密码登录查看是否能看到节点信息

![image-20230825162815575](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162815575.png)