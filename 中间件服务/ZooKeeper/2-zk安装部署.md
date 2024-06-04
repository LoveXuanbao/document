## 一、单机安装

### 1.1、依赖准备

- 官方依赖介绍：https://zookeeper.apache.org/doc/r3.4.14/zookeeperAdmin.html

- JDK安装包下载地址：https://www.oracle.com/java/technologies/downloads/#java8

- zk安装包下载地址：https://zookeeper.apache.org/releases.html

### 1.2、JDK 安装

- 如果服务器能联网，可以在服务器上直接使用使用下面命令下载 JDK 安装包，或者按照 1.1 依赖准备 JDK 安装包下载地址下载即可

```bash
# JDK:
wget --no-check-certificate --no-cookies \
    --header "Cookie: oraclelicense=accept-securebackup-cookie"  \
    http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
```

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

### 1.3、ZooKeeper 安装

- 如果服务器能联网，可以在服务器上直接使用使用下面命令下载 zk 安装包，或者按照 1.1 依赖准备 zk 安装包下载地址下载即可

```bash
wget https://downloads.apache.org/zookeeper/zookeeper-3.8.3/apache-zookeeper-3.8.3-bin.tar.gz
```

- 创建目录及解压

```bash
# 创建 zk 安装和数据目录
mkdir -p /data1/zookeeper/zkdata 

# 解压安装包到安装目录
tar -zxvf apache-zookeeper-3.8.3-bin.tar.gz -C /data1/zookeeper/

# 
echo "1" > /data1/zookeeper/zkdata/myid

# 拷贝配置文件
cd /data1/zookeeper/apache-zookeeper-3.8.3-bin/conf/
cp zoo_sample.cfg zoo.cfg
```

- 修改**zoo.cfg**

```bash
[17:05:52 root@localhost conf] cat zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data1/zookeeper/zkdata  # 修改为我们自己创建的数据目录
clientPort=2181
maxClientCnxns=60
autopurge.snapRetainCount=3
autopurge.purgeInterval=1
#server.1=192.168.0.87:2888:3888
#server.2=192.168.0.88:2888:3888
#server.3=192.168.0.89:2888:3888
```

- 配置**zookeeper.service**

```bash
[Unit]
Description=zookeeper service
After=network.target

[Service]
User=root
Group=root
Type=forking
#Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_191"
ExecStart=/data1/zookeeper/apache-zookeeper-3.8.3-bin/bin/zkServer.sh start
ExecStop=/data1/zookeeper/apache-zookeeper-3.8.3-bin/bin/zkServer.sh stop
PrivateTmp=false
Restart=always

[Install]
WantedBy=multi-user.target
```

- 启动zk

```bash
systemctl enable zookeeper --now
```

