# 一、环境准备

## 1.1、服务器配置

| 服务器IP      | hostname     | 操作系统   | CPU  | 内存 | 磁盘 |
| ------------- | ------------ | ---------- | ---- | ---- | ---- |
| 10.202.43.46  | k8s-master-1 | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.55  | k8s-master-2 | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.56  | k8s-master-3 | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.114 | k8s-node-1   | CentOS 7.8 | 4C   | 8G   | 200G |
| 10.202.43.191 | k8s-node-2   | CentOS 7.8 | 4C   | 8G   | 200G |

## 1.2、服务器初始化

### 1.2.1、所有主机分别修改hostname

```shell
hostnamectl  set-hostname k8s-master-1
hostnamectl  set-hostname k8s-master-2
hostnamectl  set-hostname k8s-master-3
hostnamectl  set-hostname k8s-node-1
hostnamectl  set-hostname k8s-node-1
```

### 1.2.2、IP和主机名映射

> `repository.test.com`为`nexus`镜像仓库的域名

```shell
cat >> /etc/hosts <<EOF
10.202.43.46  k8s-master-1
10.202.43.55  k8s-master-2
10.202.43.56  k8s-master-3
10.202.43.114 k8s-node-1
10.202.43.191 k8s-node-2
10.202.43.110 repository.test.com
EOF
```

### 1.2.3、所有节点关闭防火墙

```shell
systemctl stop firewalld  && systemctl disable firewalld

# 或者使用如下命令也可以关闭并禁用自启动
systemctl disable --now firewalld
```

### 1.2.4、所有节点关闭dnsmasq

```bash
systemctl disable --now dnsmasq
```

### 1.2.5、所有节点关闭NetworkManager

```bash
systemctl disable --now NetworkManager
```

### 1.2.6、所有节点关闭Selinux

```shell
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config && setenforce 0
```

### 1.2.7、所有节点关闭swap分区

```shell
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab && swapoff -a && sysctl -w vm.swappiness=0
```

### 1.2.8、所有节点配置ulimit

```bash
ulimit -SHn 65535

cat >> /etc/security/limits.conf <<EOF
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* seft memlock unlimited
* hard memlock unlimitedd
EOF
```

### 配置免密登录

```bash
# 安装sshpass
yum install -y sshpass

# 生成公私钥文件
ssh-keygen -f /root/.ssh/id_rsa -P ''

# 编辑sshpass脚本文件
vim sshpass.sh
#!/bin/bash
#

# 修改ip为集群IP，每个IP中间以空格隔开
export IP="10.202.43.46 10.202.43.55 10.202.43.56 10.202.43.114 10.202.43.191"
export SSHPASS=111111
for HOST in $IP;do
     sshpass -e ssh-copy-id -o StrictHostKeyChecking=no $HOST
done

# 执行脚本，配置免密
sh sshpass.sh
```

### 升级内核

内核下载网站

 http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/

 http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/

```bash
# 将下载的内核包上传至master-1服务器的/tmp目录下
cd /tmp && ll
total 74928
-rw-r--r--. 1 root root 61791728 Mar 10 16:33 kernel-ml-5.19.9-1.el7.elrepo.x86_64.rpm
-rw-r--r--. 1 root root 14930272 Mar 10 16:33 kernel-ml-devel-5.19.9-1.el7.elrepo.x86_64.rpm

# 编辑分发脚本
for HOST in k8s-master-2  k8s-master-3 k8s-node-1 k8s-node-2; do
    scp /tmp/kernel-ml* $HOST:/tmp;
done

# 所有节点安装kernel，或者使用rpm -Uvh指定文件安装
cd /tmp && yum install -y kernel-ml-*

rpm -Uvh /tmp/kernel-ml-5.19.9-1.el7.elrepo.x86_64.rpm
rpm -Uvh /tmp/kernel-ml-devel-5.19.9-1.el7.elrepo.x86_64.rpm

# 查看主机已安装内核
rpm -qa | grep kernel
kernel-3.10.0-1127.el7.x86_64
kernel-tools-3.10.0-1127.el7.x86_64
kernel-ml-devel-5.19.9-1.el7.elrepo.x86_64
kernel-ml-5.19.9-1.el7.elrepo.x86_64
kernel-tools-libs-3.10.0-1127.el7.x86_64

# 查看默认内核
grubby --default-kernel
/boot/vmlinuz-3.10.0-1127.el7.x86_64   # 安装完，使用的还是3.10的内核

# 设置使用新内核
grub2-set-default 0 && grub2-mkconfig -o /boot/grub2/grub.cfg
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"

# 查看默认任何是否修改成功
grubby --default-kernel
/boot/vmlinuz-5.19.9-1.el7.elrepo.x86_64

# 所有节点重启使用新内核
reboot
```

### 所有节点安装ipvsadm

- 安装ipvsadm和依赖

```bash
yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

- 加载模块

```bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_sh
modprobe -- br_netfilter
modprobe -- nf_conntrack
```

- 写入开机自动加载ipvsadm模块

```bash
cat > /etc/modules-load.d/k8s-modules.conf <<EOF
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
EOF
```

- 重启systemd-modules-load.service服务

```bash
systemctl restart systemd-modules-load && systemctl enable systemd-modules-load
```

- 检查是否加载成功

```bash
lsmod | grep --color=auto -e ip_vs -e nf_conntrack
ip_vs_ftp              16384  0 
nf_nat                 49152  1 ip_vs_ftp
ip_vs_sed              16384  0 
ip_vs_nq               16384  0 
ip_vs_fo               16384  0 
ip_vs_dh               16384  0 
ip_vs_lblcr            16384  0 
ip_vs_lblc             16384  0 
ip_vs_wrr              16384  0 
ip_vs_wlc              16384  0 
ip_vs_lc               16384  0 
ip_vs_sh               16384  0 
ip_vs_rr               16384  0 
ip_vs                 163840  24 ip_vs_wlc,ip_vs_rr,ip_vs_dh,ip_vs_lblcr,ip_vs_sh,ip_vs_fo,ip_vs_nq,ip_vs_lblc,ip_vs_wrr,ip_vs_lc,ip_vs_sed,ip_vs_ftp
nf_conntrack          163840  2 nf_nat,ip_vs
nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
nf_defrag_ipv4         16384  1 nf_conntrack
libcrc32c              16384  4 nf_conntrack,nf_nat,xfs,ip_vs
```

### 所有节点开启k8s集群中必须的内核参数

```bash
# 注意: /etc/sysctl.conf文件内的配置比此处的优先级高
cat > /etc/sysctl.d/k8s.conf <<EOF
net.ipv4.tcp_keepalive_time=600
net.ipv4.tcp_keepalive_intvl=30
net.ipv4.tcp_keepalive_probes=10
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv4.neigh.default.gc_stale_time=120
net.ipv4.conf.all.rp_filter=0 # 默认为1，系统会严格校验数据包的反向路径，可能导致丢包
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.default.arp_announce=2
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_announce=2
net.ipv4.ip_local_port_range= 45001 65000
net.ipv4.ip_forward=1
net.ipv4.tcp_max_tw_buckets=6000
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_synack_retries=2
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.netfilter.nf_conntrack_max=2310720
net.core.netdev_max_backlog=10000 # 每CPU网络设备积压队列长度
net.core.rmem_max = 16777216 # 所有协议类型读写的缓存区大小
net.core.wmem_max = 16777216
net.ipv4.tcp_max_syn_backlog = 4096 # 第一个积压队列长度
net.core.somaxconn = 1024 # 第二个积压队列长度
fs.inotify.max_user_instances=8192 # 表示每一个real user ID可创建的inotify instatnces的数量上限，默认128.
fs.inotify.max_user_watches=524288 # 同一用户同时可以添加的watch数目，默认8192。
fs.file-max=52706963
fs.nr_open=52706963
kernel.pid_max = 4194303
net.bridge.bridge-nf-call-arptables=1
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
vm.max_map_count = 262144
EOF
```

- 使配置生效

```bash
sysctl --system
```

### 所有节点优化日志

- 创建日志目录

```bash
mkdir -p /var/log/journal
mkdir -p /etc/systemd/journald.conf.d
```

- 配置修改日志配置文件

```bash
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent
# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000
# 最大占用空间 10G
SystemMaxUse=3G
# 单日志文件最大 200M
SystemMaxFileSize=100M
# 日志保存时间 2 周
MaxRetentionSec=2week
# 不将日志转发到 syslog
ForwardToSyslog=no
EOF
```

- 重启systedm-journald服务

```bash
systemctl restart systemd-journald && systemctl enable systemd-journald
```

### 所有节点安装docker容器运行时（二选一）

- 安装docker-ce-20.10.8

```bash
# 使用yum安装
yum install -y docker-ce-20.10.8

# 使用安装包安装，上传docker-20.tar.gz到所有服务器的/tmp目录下
cd /tmp && tar -zxf docker-20.tar.gz
cd docker && yum install -y ./*.rpm
```

- 创建docker数据目录和配置文件目录

```bash
mkdir -p /data/docker_customized /etc/docker/
```

- 添加docker配置文件
  - graph: 指定docker数据目录
  - storage-driver:  要使用的存储驱动程序
  - max-concurrent-downloads: 拉取镜像最大并发下载，默认3
  - max-concurrent-uploads: 镜像最大并发上传，默认5
  - log-driver: 容器日志的驱动程序，默认json-file
  - live-restore: 在dockerd停止时保证已启动的running容器持续运行，并在daemon进程启动后重启接管
  - log-opts: 日志配置，max-size，设置一个容器日志的大小，max-file，设置容器日志有几个文件

```bash
cat > /etc/docker/daemon.json <<EOF
{
    "graph": "/data/docker_customized",
    "storage-driver": "overlay2",
    "max-concurrent-downloads": 10,
    "max-concurrent-uploads": 5,
    "log-driver": "json-file",
    "exec-opts": ["native.cgroupdriver=systemd"],
    "live-restore": true,
    "log-opts": {
      "max-size": "500m",
      "max-file": "3"
    },
    "insecure-registries": [
           "law-harbor.internal.gridsumdissector.com",
           "harbor.service.moebius.com",
           "repository.test.com:8444",
           "repository.test.com:8445"
     ]
}
EOF
```

- 重启docker并添加开机自启动

```bash
systemctl enable --now docker
```

### 所有节点安装containerd容器运行时（二选一）

- 创建cni插件目录

```bash
mkdir -p /opt/cni/bin
mkdir -p /etc/cni/net.d
```

- 下载

```bash
# containerd 下载地址
https://github.com/containerd/containerd/tags

# 插件下载地址
https://github.com/containernetworking/plugins/tags


wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
wget https://github.com/containerd/containerd/releases/download/v1.4.13/cri-containerd-cni-1.4.13-linux-amd64.tar.gz
```

从中选择对应的版本下载，然后上传到k8s-master-1服务器上/tmp目录下

- 在k8s-master-1服务器上执行命令，拷贝到其他节点

```bash
for i in k8s-master-2 k8s-master-3 k8s-node-1 k8s-node-2; do scp /tmp/cni* /tmp/cri* $i:/tmp; done
```

- 解压安装包中内容到对应目录

```bash
tar -zxvf /tmp/cni-plugins-linux-amd64-v1.1.1.tgz -C /opt/cni/bin/
tar -zxvf /tmp/cri-containerd-cni-1.4.13-linux-amd64.tar.gz -C /
```

- 所有节点配置Containerd所需要的模块

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

- 所有节点加载模块

```bash
modprobe -- overlay
modprobe -- br_netfilter
systemctl restart systemd-modules-load.service
```

- 所有节点配置Containerd所需的内核

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

- 所有节点配置Containerd配置文件

```bash
mkdir -p /etc/containerd

# 创建默认配置文件
containerd config default | tee /etc/containerd/config.toml

# 修改
sed -i 's#SystemdCgroup = false#SystemdCgroup = true#g' /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup

sed -i "s#k8s.gcr.io/pause:3.2#repository.test.com:8444/kubernetes/containers/pause:3.7#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep sandbox_image
```

- 创建启动文件

```bash
cat > /etc/systemd/system/containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF
```

- 启动并设为开机自自动

```bash
systemctl daemon-reload
systemctl enable --now containerd
```









# k8s基本组件安装

## Master节点安装

### master-1节点下载kubernetes基本组件

#### 下载kubernetes安装包

- 第一种方法：在gitlab中找到对应版本下载

```bash
https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG
```

选择一个版本进去

![image-20230825164850095](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164850095.png)

点击 Server Binaries

![image-20230825164929954](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164929954.png)

点击 kubernetes-server-linux-amd64.tar.gz 下载

![image-20230825164958374](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825164958374.png)

- 第二种方法：直接在服务器执行下面命令下载（需要联通外网，其他版本需修改v1.18.20为其他版本号）

```bash
wget https://dl.k8s.io/v1.18.20/kubernetes-server-linux-amd64.tar.gz
```

#### 解压kubernetes安装文件

- 将压缩包内的指定文件解压到/usr/local/bin目录下

```bash
tar -zxf /tmp/kubernetes-server-linux-amd64.tar.gz --strip-components=3 \
 -C /usr/local/bin kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy}
```

- 查看解压后的文件

```bash
ll /usr/local/bin/kube*
-rwxr-xr-x 1 root root 120627200 May 12  2021 /usr/local/bin/kube-apiserver
-rwxr-xr-x 1 root root 110022656 May 12  2021 /usr/local/bin/kube-controller-manager
-rwxr-xr-x 1 root root  43962368 May 12  2021 /usr/local/bin/kubectl
-rwxr-xr-x 1 root root 113292152 May 12  2021 /usr/local/bin/kubelet
-rwxr-xr-x 1 root root  38326272 May 12  2021 /usr/local/bin/kube-proxy
-rwxr-xr-x 1 root root  43737088 May 12  2021 /usr/local/bin/kube-scheduler
```

- 分发kubernetes相关文件到其他master节点

```bash
vim /tmp/cp_kubernetes_install.sh
#!/bin/bash
#

cd /usr/local/bin/
for NODE in k8s-master-2 k8s-master-3; do
    for FILE in kube-apiserver kube-controller-manager kubectl kube-scheduler; do
        scp /usr/local/bin/$FILE $NODE:/usr/local/bin/${FILE}
    done
done


# 赋予执行权限
chmod +x /tmp/cp_kubernetes_install.sh
# 执行脚本
sh /tmp/cp_kubernetes_install.sh
```

#### 下载etcd安装包

- 第一种办法：在gitlab找到对应的releases下载

```bash
https://github.com/etcd-io/etcd/releases
```

找到对应版本下的Assets，选择linux-adm64的下载

![image-20230825165041087](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825165041087.png)

- 第二种办法：直接在服务器执行下面命令下载（需要联通外网，其他版本需修改对版本号为其他版本号）

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.4.16/etcd-v3.4.16-linux-amd64.tar.gz
```

### master-1节点下载证书生成工具 

- 使用安装包

```bash
# 上传cfssl_1.6.3.tgz至master-1服务器
# 解压
tar -zxvf cfssl_1.6.3.tgz
cd cfssl_1.6.3
cp cfssl_1.6.3_linux_amd64 /usr/local/bin/cfssl
cp cfssljson_1.6.3_linux_amd64 /usr/local/bin/cfssljson
cp cfssl-certinfo_1.6.3_linux_amd64 /usr/local/bin/cfssl-certinfo
```

- gitlab下载

```bash
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl_1.6.3_linux_amd64 -O /usr/local/bin/cfssl
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssljson_1.6.3_linux_amd64 -O /usr/local/bin/cfssljson
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl-certinfo_1.6.3_linux_amd64 -O /usr/local/bin/cfssl-certinfo
```

- pkg.cfssl.org下载

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/local/bin/cfssl
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/local/bin/cfssljson
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/local/bin/cfssl-certinfo
```

### master-1节点创建证书目录

生成证书操作都在master-1节点执行

```bash
mkdir -p /opt/pki               # 创建生成证书文件目录
mkdir -p /etc/etcd/ssl          # 创建etcd证书目录
mkdir -p /etc/kubernetes/pki    # 创建kubernetes证书目录
```

- 创建ca-config.json
  - server 服务端密钥证书
  - client   客户端密钥证书
  - peer   双向认证

```bash
cat > /opt/pki/ca-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "175200h"
        },
        "profiles": {
            "server": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF
```

- 创建ca-csr.json

```bash
cat > /opt/pki/ca-csr.json   << EOF 
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "Kubernetes",
      "OU": "Kubernetes-manual"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
EOF
```

#### 生成etcd证书

- 创建生成证书的csr（证书签名请求）文件

```bash
cat > /opt/pki/etcd-ca-csr.json  << EOF 
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
EOF
```

- 通过csr文件生成etcd的ca证书和ca证书的key
  - ca证书用来颁发etcd的客户端证书，相当于某证书机构

```bash
cd /opt/pki
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca
```

- 查看生成的证书

```bash
ll /etc/etcd/ssl/
total 24
-rw-r--r-- 1 root root 1050 Mar 11 10:24 etcd-ca.csr        # etcd的ca证书的csr
-rw------- 1 root root 1679 Mar 11 10:31 etcd-ca-key.pem    # etcd的ca证书的key
-rw-r--r-- 1 root root 1318 Mar 11 10:31 etcd-ca.pem        # etcd的ca证书
```

- 创建etcd证书的csr

```bash
cat > /opt/pki/etcd-csr.json <<EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ]
}
EOF
```

- 根据一级ca证书，生成etcd证书；注意：如果有后期扩容计划，最后在hostname中多写几个IP预留出来
  - -ca=指定一级ca证书
  - -ca-key=指定一级ca证书的key
  - -config=指定配置文件
  - -hostname=指定信任的主机
  - -profile=选择配置文件里的双向认证的配置

```bash
cfssl gencert \
   -ca=/etc/etcd/ssl/etcd-ca.pem \
   -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
   -config=ca-config.json \
   -hostname=127.0.0.1,k8s-master-1,k8s-master-2,k8s-master-3,10.202.43.46,10.202.43.55,10.202.43.56 \
   -profile=peer \
   etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
```

- 查看生成的etcd证书

```bash
ll /etc/etcd/ssl/
total 24
-rw-r--r-- 1 root root 1050 Mar 10 19:02 etcd-ca.csr
-rw------- 1 root root 1679 Mar 10 19:02 etcd-ca-key.pem
-rw-r--r-- 1 root root 1318 Mar 10 19:02 etcd-ca.pem
-rw-r--r-- 1 root root 1074 Mar 10 19:28 etcd.csr
-rw------- 1 root root 1679 Mar 10 19:28 etcd-key.pem
-rw-r--r-- 1 root root 1403 Mar 10 19:28 etcd.pem
```

#### 生成kubernetes的ca证书

- 创建kubernetes的一级证书

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca
```

- 查看kubernetes的ca证书

```bash
ll /etc/kubernetes/pki/
total 12
-rw-r--r-- 1 root root 1070 Mar 11 11:18 ca.csr
-rw------- 1 root root 1675 Mar 11 11:18 ca-key.pem
-rw-r--r-- 1 root root 1363 Mar 11 11:18 ca.pem
```

#### 生成kubernetes的apiserver证书

- 创建apiserver-csr文件

```bash
cat > /opt/pki/kube-apiserver-csr.json << EOF 
{
  "CN": "kube-apiserver",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "Kubernetes",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```

- 根据一级ca证书，生成apiserver证书
  - 172.168.0.1为svc的第一个ip
  - 10.202.43.192为VIP

```bash
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json  \
    -hostname=172.168.0.1,10.202.43.192,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,10.202.43.46,10.202.43.55,10.202.43.56 \
    -profile=peer \
    apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver
```

#### 生成apiserver的聚合证书

- 创建front-proxy-ca-csr文件

```bash
cat > /opt/pki/kube-front-proxy-ca-csr.json  << EOF 
{
  "CN": "kubernetes",
  "key": {
     "algo": "rsa",
     "size": 2048
  },
  "ca": {
    "expiry": "876000h"
  }
}
EOF
```

- 根据front-proxy-ca-csr.json文件生成front-proxy一级证书

```bash
cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca
```

- 查看front-proxy一级证书

```bash
ll /etc/kubernetes/pki/front-proxy-ca*
-rw-r--r-- 1 root root  940 Mar 11 11:28 /etc/kubernetes/pki/front-proxy-ca.csr
-rw------- 1 root root 1679 Mar 11 11:28 /etc/kubernetes/pki/front-proxy-ca-key.pem
-rw-r--r-- 1 root root 1094 Mar 11 11:28 /etc/kubernetes/pki/front-proxy-ca.pem
```

- 创建front-proxy-client-csr文件

```bash
cat > /opt/pki/kube-front-proxy-client-csr.json  << EOF 
{
  "CN": "front-proxy-client",
  "key": {
     "algo": "rsa",
     "size": 2048
  }
}
EOF
```

- 根据front-proxy一级证书，生成front-proxy-client证书

```bash
cfssl gencert \
    -ca=/etc/kubernetes/pki/front-proxy-ca.pem \
    -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem \
    -config=ca-config.json \
    -profile=peer \
    front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client
    
# 返回结果（忽略警告）
2023/03/11 11:33:38 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

- 查看front-proxy-client证书

```bash
ll /etc/kubernetes/pki/front-proxy-client*
-rw-r--r-- 1 root root  903 Mar 11 11:33 /etc/kubernetes/pki/front-proxy-client.csr
-rw------- 1 root root 1675 Mar 11 11:33 /etc/kubernetes/pki/front-proxy-client-key.pem
-rw-r--r-- 1 root root 1188 Mar 11 11:33 /etc/kubernetes/pki/front-proxy-client.pem
```

#### 生成kube-controller-manager的证书

- 创建kube-controller-manager-csr文件

```bash
cat > /opt/pki/kube-controller-manager-csr.json << EOF 
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```

- 根据一级ca证书，生成kube-controller-manager证书

```bash
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=peer \
    kube-controller-manager-csr.json | cfssljson -bare /etc/kubernetes/pki/kube-controller-manager
    
# 返回结果（忽略警告）
2023/03/11 11:43:11 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

- 查看kube-controller-manager证书

```bash
ll /etc/kubernetes/pki/kube-controller-manager*
-rw-r--r-- 1 root root 1082 Mar 11 11:43 /etc/kubernetes/pki/kube-controller-manager.csr
-rw------- 1 root root 1679 Mar 11 11:43 /etc/kubernetes/pki/kube-controller-manager-key.pem
-rw-r--r-- 1 root root 1497 Mar 11 11:43 /etc/kubernetes/pki/kube-controller-manager.pem
```

#### 生成kube-controller-manager上下文文件

- set-cluster: 设置一个集群项
  - 10.202.43.192是VIP，如果不是高可用集群，改成master的IP即可
  - 8443是apiserver的端口，默认是6443

```bash
kubectl config set-cluster kubernetes \
     --certificate-authority=/etc/kubernetes/pki/ca.pem \
     --embed-certs=true \
     --server=https://10.202.43.192:8443 \
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

- set-context: 设置一个环境项，一个上下文

```bash
kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

- set-credentials: 设置一个用户项

```bash
kubectl config set-credentials system:kube-controller-manager \
     --client-certificate=/etc/kubernetes/pki/kube-controller-manager.pem \
     --client-key=/etc/kubernetes/pki/kube-controller-manager-key.pem \
     --embed-certs=true \
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

- use-context: 设置默认环境

```bash
kubectl config use-context system:kube-controller-manager@kubernetes \
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

#### 生成kube-scheduler证书

- 创建kube-scheduler-csr文件

```bash
cat > /opt/pki/kube-scheduler-csr.json << EOF 
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```

- 根据一级ca证书，生成kube-scheduler证书

```bash
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=peer \
    kube-scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/kube-scheduler
    
# 返回结果（忽略警告）
2023/03/11 12:25:02 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

- 查看kube-scheduler证书

```bash
ll /etc/kubernetes/pki/kube-scheduler*
-rw-r--r-- 1 root root 1058 Mar 11 12:25 /etc/kubernetes/pki/kube-scheduler.csr
-rw------- 1 root root 1679 Mar 11 12:25 /etc/kubernetes/pki/kube-scheduler-key.pem
-rw-r--r-- 1 root root 1472 Mar 11 12:25 /etc/kubernetes/pki/kube-scheduler.pem
```

#### 生成kube-scheduler上下文文件

- set-cluster: 设置一个集群项
  - 10.202.43.192是VIP，如果不是高可用集群，改成master的IP即可
  - 8443是apiserver的端口，默认是6443

```bash
kubectl config set-cluster kubernetes \
     --certificate-authority=/etc/kubernetes/pki/ca.pem \
     --embed-certs=true \
     --server=https://10.202.43.192:8443 \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

- set-context: 设置一个环境项，一个上下文

```bash
kubectl config set-context system:kube-scheduler@kubernetes \
     --cluster=kubernetes \
     --user=system:kube-scheduler \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

- set-credentials: 设置一个用户项

```bash
kubectl config set-credentials system:kube-scheduler \
     --client-certificate=/etc/kubernetes/pki/kube-scheduler.pem \
     --client-key=/etc/kubernetes/pki/kube-scheduler-key.pem \
     --embed-certs=true \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

- use-context: 设置默认环境

```bash
kubectl config use-context system:kube-scheduler@kubernetes \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

#### 生成admin证书

- 创建admin-csr文件

```bash
cat > /opt/pki/admin-csr.json << EOF 
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:masters",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```

- 根据一级根证书和admin-csr文件生成admin证书

```bash
cfssl gencert \
   -ca=/etc/kubernetes/pki/ca.pem \
   -ca-key=/etc/kubernetes/pki/ca-key.pem \
   -config=ca-config.json \
   -profile=peer \
   admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin
   
# 返回结果（忽略警告）
2023/03/11 12:28:44 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

#### 生成admin上下文文件

- set-cluster: 设置一个集群项
  - 10.202.43.192是VIP，如果不是高可用集群，改成master的IP即可
  - 8443是apiserver的端口，默认是6443

```bash
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://10.202.43.192:8443 \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

- set-context: 设置一个环境项，一个上下文

```bash
kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes \
    --user=kubernetes-admin \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

- set-credentials: 设置一个用户项

```bash
kubectl config set-credentials kubernetes-admin \
    --client-certificate=/etc/kubernetes/pki/admin.pem \
    --client-key=/etc/kubernetes/pki/admin-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

- use-context: 设置默认环境

```bash
kubectl config use-context kubernetes-admin@kubernetes \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

#### 生成kube-proxy证书

- 创建kube-proxy-csr文件

```bash
cat > /opt/pki/kube-proxy-csr.json  << EOF 
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "system:kube-proxy",
      "OU": "Kubernetes-manual"
    }
  ]
}
EOF
```

- 根据一级根证书和kube-proxy-csr文件生成kube-proxy证书

```bash
cfssl gencert \
    -ca=/etc/kubernetes/pki/ca.pem \
    -ca-key=/etc/kubernetes/pki/ca-key.pem \
    -config=ca-config.json \
    -profile=peer \
    kube-proxy-csr.json | cfssljson -bare /etc/kubernetes/pki/kube-proxy
```

#### 创建kube-proxy上下文

- set-cluster: 设置一个集群项
  - 10.202.43.192是VIP，如果不是高可用集群，改成master的IP即可
  - 8443是apiserver的端口，默认是6443

```bash
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://10.202.43.192:8443 \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

- set-context: 设置一个环境项，一个上下文

```bash
kubectl config set-context kube-proxy@kubernetes \
    --cluster=kubernetes \
    --user=kube-proxy \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

- set-credentials: 设置一个用户项

```bash
kubectl config set-credentials kube-proxy \
    --client-certificate=/etc/kubernetes/pki/kube-proxy.pem \
    --client-key=/etc/kubernetes/pki/kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

- use-context: 设置默认环境

```bash
kubectl config use-context kube-proxy@kubernetes \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

#### 创建ServiceAccount Key

```bash
openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
# 返回结果
Generating RSA private key, 2048 bit long modulus
.................................+++
.+++
e is 65537 (0x10001)

openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
# 返回结果
writing RSA key
```

#### 发送证书到其他master节点

```bash
vim /tmp/cp_kubernetes_ssl.sh
#!/bin/bash
#

for NODE in k8s-master-2 k8s-master-3; do
    ssh $NODE "mkdir -p /etc/kubernetes/pki"
    for FILE in $(ls /etc/kubernetes/pki/ | grep -v etcd); do
      scp /etc/kubernetes/pki/${FILE} $NODE:/etc/kubernetes/pki/${FILE}
    done

    for FILE in $(ls /etc/kubernetes/ | grep -v pki); do
      scp /etc/kubernetes/${FILE} $NODE:/etc/kubernetes/${FILE}
    done
done


# 执行脚本
sh /tmp/cp_kubernetes_ssl.sh
```

### 三台master节点安装etcd

- 分发安装包到其他两台master节点

```bash
for HOST in k8s-master-2  k8s-master-3; do scp /tmp/etcd-v3.4.16-linux-amd64.tar.gz $HOST:/tmp; done
```

- 解压etcd安装包

```bash
cd /tmp && tar -zxvf etcd-v3.4.16-linux-amd64.tar.gz --strip-components=1 \
-C /usr/local/bin etcd-v3.4.16-linux-amd64/etcd{,ctl}
```

- 将etcd证书分发到其他节点

```bash
vim cp_etcd_ssl.sh
#!/bin/bash
#

etcd='k8s-master-2 k8s-master-3'
for NODE in $etcd; do
    ssh $NODE "mkdir -p /etc/etcd/ssl"
    for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
        scp /etc/etcd/ssl/${FILE} $NODE:/etc/etcd/ssl/${FILE}
    done
done

# 执行脚本
sh /tmp/cp_etcd_ssl.sh
```

#### k8s-master-1配置

```bash
cat > /etc/etcd/etcd.config.yml << EOF 
name: 'etcd0'
data-dir: /data1/etcd
wal-dir: /data1/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://10.202.43.46:2380'
listen-client-urls: 'https://10.202.43.46:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://10.202.43.46:2380'
advertise-client-urls: 'https://10.202.43.46:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'etcd0=https://10.202.43.46:2380,etcd1=https://10.202.43.55:2380,etcd2=https://10.202.43.56:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false
EOF
```

#### k8s-master-2配置

```bash
cat > /etc/etcd/etcd.config.yml << EOF 
name: 'etcd1'
data-dir: /data1/etcd
wal-dir: /data1/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://10.202.43.55:2380'
listen-client-urls: 'https://10.202.43.55:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://10.202.43.55:2380'
advertise-client-urls: 'https://10.202.43.55:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'etcd0=https://10.202.43.46:2380,etcd1=https://10.202.43.55:2380,etcd2=https://10.202.43.56:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false
EOF
```

#### k8s-master-3配置

```bash
cat > /etc/etcd/etcd.config.yml << EOF 
name: 'etcd2'
data-dir: /data1/etcd
wal-dir: /data1/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://10.202.43.56:2380'
listen-client-urls: 'https://10.202.43.56:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://10.202.43.56:2380'
advertise-client-urls: 'https://10.202.43.56:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'etcd0=https://10.202.43.46:2380,etcd1=https://10.202.43.55:2380,etcd2=https://10.202.43.56:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/etcd/ssl/etcd.pem'
  key-file: '/etc/etcd/ssl/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/etcd/ssl/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false
EOF
```

#### 所有master节点创建etcd.service启动文件

```bash
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd3.service
EOF
```

#### 重启etcd并添加开机自启动

```bash
systemctl daemon-reload && systemctl enable --now etcd
```

#### 查看etcd状态

```bash
export ETCDCTL_API=3
etcdctl \
    --endpoints="10.202.43.46:2379,10.202.43.55:2379,10.202.43.56:2379" \
    --cacert=/etc/etcd/ssl/etcd-ca.pem \
    --cert=/etc/etcd/ssl/etcd.pem \
    --key=/etc/etcd/ssl/etcd-key.pem \
    endpoint status --write-out=table
    
# 返回结果如下代表etcd集群正常
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|     ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.202.43.46:2379 | 8afc7437e67733e3 |  3.4.16 |   20 kB |      true |      false |         3 |          9 |                  9 |        |
| 10.202.43.55:2379 | d3835aa58a16c295 |  3.4.16 |   20 kB |     false |      false |         3 |          9 |                  9 |        |
| 10.202.43.56:2379 | f9567391c093655a |  3.4.16 |   20 kB |     false |      false |         3 |          9 |                  9 |        |
+-------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

### 三台master节点安装keepalive和haproxy

如果规划的有单独的keepalived和haproxy主机，在其规划节点安装即可，本次实验没有额外规划，所以在master节点安装keepalived和haproxy服务；

如果单独规划主机安装keepalived和haproxy，两台主机即可

#### 安装keepalived和haproxy

```bash
yum install -y keepalived haproxy
```

#### 配置haproxy

- 备份源配置文件

```bash
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
```

- 配置haproxy.cfg，三台master节点配置一样

```bash
cat >/etc/haproxy/haproxy.cfg<<"EOF"
global
 maxconn 2000
 ulimit-n 16384
 log 127.0.0.1 local0 err
 stats timeout 30s

defaults
 log global
 mode http
 option httplog
 timeout connect 5000
 timeout client 50000
 timeout server 50000
 timeout http-request 15s
 timeout http-keep-alive 15s

frontend k8s-master
 bind 0.0.0.0:8443
 bind 127.0.0.1:8443
 mode tcp
 option tcplog
 tcp-request inspect-delay 5s
 default_backend k8s-master


backend k8s-master
 mode tcp
 option tcplog
 option tcp-check
 balance roundrobin
 default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
 server  k8s-master01  10.202.43.46:6443 check
 server  k8s-master02  10.202.43.55:6443 check
 server  k8s-master03  10.202.43.56:6443 check
EOF
```

#### 配置keepalived

- 备份源配置文件，三台master都要执行

```bash
cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
```

- master-1配置keepalived.conf

```bash
cat > /etc/keepalived/keepalived.conf << EOF
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5 
    weight -5
    fall 2
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    # 注意网卡名
    interface eth0 
    mcast_src_ip 10.202.43.46
    virtual_router_id 88
    priority 120
    nopreempt
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        10.202.43.192
    }
    track_script {
      chk_apiserver 
    }
}
EOF
```

- master-2\master-3配置keepalived.conf

```bash
cat > /etc/keepalived/keepalived.conf << EOF
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5 
    weight -5
    fall 2
    rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    # 注意网卡名
    interface eth0
	# 注意修改ip为当前节点IP
    mcast_src_ip 10.202.43.56
    virtual_router_id 88
    priority 100
    nopreempt
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        10.202.43.192
    }
    track_script {
      chk_apiserver 
    }
}
EOF
```

#### 健康检查脚本配置

- 三台keepalived节点都要配置

```bash
cat >  /etc/keepalived/check_apiserver.sh << EOF
#!/bin/bash

err=0
for k in \$(seq 1 3)
do
    check_code=\$(pgrep haproxy)
    if [[ \$check_code == "" ]]; then
        err=\$(expr \$err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ \$err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
EOF
```

- 给脚本授可执行权限

```bash
chmod +x /etc/keepalived/check_apiserver.sh
```

#### 启动haproxy和keepalived

```bash
systemctl daemon-reload
systemctl enable --now haproxy
systemctl enable --now keepalived
```

### kubernetes组件配置

所有master节点创建相关目录

```bash
mkdir -p /etc/kubernetes/manifests/ \
    /etc/systemd/system/kubelet.service.d \
    /var/lib/kubelet \
    /var/log/kubernetes
```

#### 配置apiserver启动文件

- k8s-master-1

```bash
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
      --v=2 \\
      --logtostderr=true \\
      --allow-privileged=true \\
      --bind-address=0.0.0.0 \\
      --secure-port=6443 \\
      --advertise-address=10.202.43.46 \\
      --service-cluster-ip-range=172.10.0.0/16 \\
      --service-node-port-range=30000-32767 \\
      --etcd-servers=https://10.202.43.46:2379,https://10.202.43.55:2379,https://10.202.43.56:2379 \\
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem \\
      --etcd-certfile=/etc/etcd/ssl/etcd.pem \\
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\
      --client-ca-file=/etc/kubernetes/pki/ca.pem \\
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem \\
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \\
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem \\
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem \\
      --service-account-key-file=/etc/kubernetes/pki/sa.pub \\
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key \\
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \\
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota \\
      --authorization-mode=Node,RBAC \\
      --enable-bootstrap-token-auth=true \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \\
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem \\
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem \\
      --requestheader-allowed-names=aggregator \\
      --requestheader-group-headers=X-Remote-Group \\
      --requestheader-extra-headers-prefix=X-Remote-Extra- \\
      --requestheader-username-headers=X-Remote-User \\
      --enable-aggregator-routing=true
      # --feature-gates=IPv6DualStack=true
      # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- k8s-master-2

```bash
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
      --v=2 \\
      --logtostderr=true \\
      --allow-privileged=true \\
      --bind-address=0.0.0.0 \\
      --secure-port=6443 \\
      --advertise-address=10.202.43.55 \\
      --service-cluster-ip-range=172.10.0.0/16 \\
      --service-node-port-range=30000-32767 \\
      --etcd-servers=https://10.202.43.46:2379,https://10.202.43.55:2379,https://10.202.43.56:2379 \\
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem \\
      --etcd-certfile=/etc/etcd/ssl/etcd.pem \\
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\
      --client-ca-file=/etc/kubernetes/pki/ca.pem \\
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem \\
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \\
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem \\
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem \\
      --service-account-key-file=/etc/kubernetes/pki/sa.pub \\
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key \\
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \\
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota \\
      --authorization-mode=Node,RBAC \\
      --enable-bootstrap-token-auth=true \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \\
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem \\
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem \\
      --requestheader-allowed-names=aggregator \\
      --requestheader-group-headers=X-Remote-Group \\
      --requestheader-extra-headers-prefix=X-Remote-Extra- \\
      --requestheader-username-headers=X-Remote-User \\
      --enable-aggregator-routing=true
      # --feature-gates=IPv6DualStack=true
      # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- k8s-master-3

```bash
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
      --v=2 \\
      --logtostderr=true \\
      --allow-privileged=true \\
      --bind-address=0.0.0.0 \\
      --secure-port=6443 \\
      --advertise-address=10.202.43.56 \\
      --service-cluster-ip-range=172.10.0.0/16 \\
      --service-node-port-range=30000-32767 \\
      --etcd-servers=https://10.202.43.46:2379,https://10.202.43.55:2379,https://10.202.43.56:2379 \\
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem \\
      --etcd-certfile=/etc/etcd/ssl/etcd.pem \\
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\
      --client-ca-file=/etc/kubernetes/pki/ca.pem \\
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem \\
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \\
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem \\
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem \\
      --service-account-key-file=/etc/kubernetes/pki/sa.pub \\
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key \\
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \\
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota \\
      --authorization-mode=Node,RBAC \\
      --enable-bootstrap-token-auth=true \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \\
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem \\
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem \\
      --requestheader-allowed-names=aggregator \\
      --requestheader-group-headers=X-Remote-Group \\
      --requestheader-extra-headers-prefix=X-Remote-Extra- \\
      --requestheader-username-headers=X-Remote-User \\
      --enable-aggregator-routing=true
      # --feature-gates=IPv6DualStack=true
      # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

#### 启动apiserver服务

- 所有master节点

```bash
systemctl daemon-reload
systemctl enable --now kube-apiserver
```

#### 配置kube-controller-manager启动文件

- 所有master节点都配置

```bash
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
      --v=2 \\
      --logtostderr=true \\
      --bind-address=127.0.0.1 \\
      --root-ca-file=/etc/kubernetes/pki/ca.pem \\
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \\
      --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \\
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key \\
      --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \\
      --leader-elect=true \\
      --use-service-account-credentials=true \\
      --node-monitor-grace-period=40s \\
      --node-monitor-period=5s \\
      --pod-eviction-timeout=2m0s \\
      --controllers=*,bootstrapsigner,tokencleaner \\
      --allocate-node-cidrs=true \\
      --cluster-cidr=172.20.0.0/12 \\
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \\
      --node-cidr-mask-size=24
      # --cluster-signing-duration=876000h0m0s \\   k8s-v1.18.20之前不支持改参数
      # --node-cidr-mask-size-ipv4=24
      # --service-cluster-ip-range=172.168.0.0/16
      # --node-cidr-mask-size-ipv6=64
      # --feature-gates=IPv6DualStack=true

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

#### 启动kube-controller-manager服务

- 所有master节点

```bash
systemctl daemon-reload
systemctl enable --now kube-controller-manager
```

#### 配置kube-scheduler启动文件

- 所有master节点

```bash
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
      --v=2 \\
      --logtostderr=true \\
      --bind-address=127.0.0.1 \\
      --leader-elect=true \\
      --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

#### 启动kube-scheduler服务

- 所有master节点

```bash
systemctl daemon-reload
systemctl enable --now kube-scheduler
```

### TLS Bootstrapping配置

#### 生成bootstrap-kubelet上下文文件

- 在k8s-master-1上配置

- set-cluster: 设置一个集群项
  - 10.202.43.192是VIP，如果不是高可用集群，改成master的IP即可
  - 8443是apiserver的端口，默认是6443

```bash
kubectl config set-cluster kubernetes \
    --certificate-authority=/etc/kubernetes/pki/ca.pem \
    --embed-certs=true \
    --server=https://10.202.43.192:8443 \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

- set-context: 设置一个环境项，一个上下文

```bash
kubectl config set-context tls-bootstrap-token-user@kubernetes \
    --cluster=kubernetes \
    --user=tls-bootstrap-token-user \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

- set-credentials: 设置一个用户项

```bash
kubectl config set-credentials tls-bootstrap-token-user \
    --token=c8ad9c.2e4d610cf3e7426e \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

- use-context: 设置默认环境

```bash
kubectl config use-context tls-bootstrap-token-user@kubernetes \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

#### 创建bootstrap.secret.yaml 文件

- 如果要修改token-id和token-secret，则需要和上下文中的token相对应

```bash
cat > /opt/pki/bootstrap.secret.yaml << EOF 
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-c8ad9c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  description: "The default bootstrap token generated by 'kubelet '."
  token-id: c8ad9c
  token-secret: 2e4d610cf3e7426e
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups:  system:bootstrappers:default-node-token,system:bootstrappers:worker,system:bootstrappers:ingress
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubelet-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node-bootstrapper
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-bootstrap
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:nodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:default-node-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-autoapprove-certificate-rotation
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:nodes
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kube-apiserver
EOF
```

#### 创建kubectl配置文件

```bash
mkdir -p /root/.kube; cp /etc/kubernetes/admin.kubeconfig /root/.kube/config
```

#### 创建bootstrap.secret

```bash
kubectl create -f /opt/pki/bootstrap.secret.yaml
# 返回结果如下
secret/bootstrap-token-c8ad9c created
clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-bootstrap created
clusterrolebinding.rbac.authorization.k8s.io/node-autoapprove-certificate-rotation created
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

### 查看集群状态

- 返回结果如下，继续操作，如果状态不是Healthy，请排查对应服务，解决问题后，在继续

```bash
kubectl get cs
# 返回结果如下，
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-2               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}
```

## node节点安装

#### 拷贝k8s-master-1上的证书到node节点

- 在k8s-master-1编辑以下脚本并执行

```bash
vim /tmp/cp_k8s_ssl_to_node.sh
#!/bin/bash
#

cd /etc/kubernetes/
for NODE in k8s-master-2 k8s-master-3 k8s-node-1 k8s-node-2;do
    ssh $NODE "mkdir -p /etc/kubernetes/pki"
    for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem pki/kube-proxy.csr pki/kube-proxy-key.pem pki/kube-proxy.pem kube-proxy.kubeconfig bootstrap-kubelet.kubeconfig;do
      scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/${FILE}
    done
done


# 给脚本赋予执行权限
chmod +x /tmp/cp_k8s_ssl_to_node.sh

# 执行脚本
sh /tmp/cp_k8s_ssl_to_node.sh
```

#### 拷贝node相关服务启动文件到所有node节点

- 在k8s-master-1编辑以下脚本并执行

```bash
vim /tmp/cp_k8s_node_install.sh
#!/bin/bash
#

cd /usr/local/bin/
for NODE in k8s-master-2 k8s-master-3 k8s-node-1 k8s-node-2; do
    for FILE in kubelet kube-proxy; do
        scp /usr/local/bin/$FILE $NODE:/usr/local/bin/${FILE}
    done
done


# 给脚本赋予执行权限
chmod +x /tmp/cp_k8s_node_install.sh

# 执行脚本
sh /tmp/cp_k8s_node_install.sh
```

- 所有node节点创建相关目录

```bash
mkdir -p /etc/kubernetes/manifests/ \
    /etc/systemd/system/kubelet.service.d \
    /var/lib/kubelet \
    /var/log/kubernetes
```

#### 配置kubelet.service启动文件

- 所有node节点上创建: **注意修改--hostname-override=所在nodeIP**

```bash
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
    --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig  \\
    --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \\
    --config=/etc/kubernetes/kubelet-conf.yml \\
    --container-runtime=remote \\
    --runtime-request-timeout=15m \\
    --container-runtime-endpoint=unix:///run/containerd/containerd.sock  \\
    --node-labels=node.kubernetes.io/node= \\
    --hostname-override=10.202.43.191 \\
    --network-plugin=cni \\
    --cni-conf-dir=/etc/cni/net.d \\
    --cni-bin-dir=/opt/cni/bin \\
    --tls-cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 \\
    --image-pull-progress-deadline=15m \\
    --logtostderr=true  \\
    --v=1
    #--pod-infra-container-image=repository.test.com:8444/kubernetes/containers/pause:3.7 \

[Install]
WantedBy=multi-user.target
EOF
```



#### 创建kubelet-conf.yml配置文件

- 如果更改了k8s的service网段，需要更改kubelet-conf.yml中的clusterDNS配置
- **修改address为所在node节点IP**

```bash
cat > /etc/kubernetes/kubelet-conf.yml <<EOF
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "10.202.43.191"
port: 10250
readOnlyPort: 10255
serverTLSBootstrap: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd
cgroupsPerQOS: true
clusterDNS:
- 172.168.0.2
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 20Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
eventBurst: 20
eventRecordQPS: 0
enableContentionProfiling: true
evictionHard:
  imagefs.available: 10%
  memory.available: 256Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionSoft:
  imagefs.available: "15%"
  memory.available: 512Mi
  nodefs.available: "15%"
  nodefs.inodesFree: "10%"
evictionSoftGracePeriod:
  imagefs.available: 3m
  memory.available: 1m
  nodefs.available: 3m
  nodefs.inodesFree: 1m
evictionPressureTransitionPeriod: 5m0s
evictionMaxPodGracePeriod: 30
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 80
imageGCLowThresholdPercent: 60
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 2000
kubeAPIQPS: 1000
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
podsPerCore: 10
podCIDR: "172.28.0.0/14"
nodeStatusUpdateFrequency: 4s
nodeStatusReportFrequency: 1m0s
nodeLeaseDurationSeconds: 40
oomScoreAdj: -999
podPidsLimit: 100000
registryBurst: 20
registryPullQPS: 0
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 10m
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
staticPodURL: ""
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
systemReserved:
  cpu: 250m
  ephemeral-storage: 2Gi
  memory: 512Mi
kubeReserved:
  cpu: 250m
  ephemeral-storage: 2Gi
  memory: 512Mi
enforceNodeAllocatable: ["pods"]
featureGates:
  DynamicKubeletConfig: true
  RotateKubeletClientCertificate: true
  ExpandCSIVolumes: true
  LocalStorageCapacityIsolation: false
EOF
```

#### 启动kubelet服务

- 所有node节点执行

```bash
systemctl daemon-reload
systemctl enable --now kubelet
```

#### 查看集群状态

```bash
kubectl get nodes 
NAME            STATUS     ROLES    AGE    VERSION
10.202.43.114   NotReady   <none>   104s   v1.18.19
10.202.43.191   NotReady   <none>   32s    v1.18.19
10.202.43.46    NotReady   <none>   18m    v1.18.19
10.202.43.55    NotReady   <none>   51s    v1.18.19
10.202.43.56    NotReady   <none>   27s    v1.18.19
```

#### kube-proxy安装

- 配置kube-proxy启动文件

```bash
cat >  /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
    --config=/etc/kubernetes/kube-proxy-conf.yml \\
    --logtostderr=true \\
    --v=2

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

#### 创建kube-proxy-conf.yml文件

- **注意：**配置文件中的几个IP修改为所在node节点的IP

```bash
cat > /etc/kubernetes/kube-proxy-conf.yml << EOF
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 10.202.43.46
clientConnection:
  acceptContentTypes: ""
  burst: 10
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
  qps: 5
clusterCIDR: 172.16.0.0/12
configSyncPeriod: 15m0s
conntrack:
  max: null
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
enableProfiling: false
healthzBindAddress: 10.202.43.46:10256
hostnameOverride: "10.202.43.46"
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 0s
  syncPeriod: 30s
ipvs:
  masqueradeAll: true
  minSyncPeriod: 5s
  scheduler: "rr"
  syncPeriod: 30s
kind: KubeProxyConfiguration
metricsBindAddress: 10.202.43.46:10249
mode: "ipvs"
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
udpIdleTimeout: 250ms
EOF
```

#### 启动kube-proxy服务

```bash
systemctl daemon-reload
systemctl enable --now kube-proxy
```

### 创建docker-secret

```bash
kubectl create secret docker-registry my-secret --docker-server=repository.test.com:8444 --docker-username=admin --docker-password=admin123 -n kube-system
```

## 安装Calico

### 下载对应的yaml文件

- 通过网站查看calico版本，修改对应的版本下载
  - **https://docs.projectcalico.org**
- 在一台能连外网的服务器，下载calico.yaml文件

```bash
wget https://docs.projectcalico.org/archive/v3.20/manifests/calico.yaml
```

- 过滤出所用的镜像

```bash
grep -r "image:" calico.yaml
          image: docker.io/calico/cni:v3.20.6
          image: docker.io/calico/cni:v3.20.6
          image: docker.io/calico/pod2daemon-flexvol:v3.20.6
          image: docker.io/calico/node:v3.20.6
          image: docker.io/calico/kube-controllers:v3.20.6
```

- 在一台外网服务器，下载以上docker镜像，上传到集群的仓库中
  - **repository.test.com:8444**是我们k8s集群的镜像仓库

```bash
docker pull docker.io/calico/cni:v3.20.6
docker tag docker.io/calico/cni:v3.20.6 repository.test.com:8444/calico/cni:v3.20.6
docker push repository.test.com:8444/calico/cni:v3.20.6
```

- 修改calico.yaml文件中的镜像地址

```bash
sed -i 's@docker.io@repository.test.com:8444@g' calico.yaml
```

- 在calicao.yaml中添加docker认证，在containers上方添加，和containers平级

```bash
      ...其他内容
      imagePullSecrets:
      - name: my-secret
      containers:
      ...其他内容
```

### 启动Calico

- 在k8s-master-1节点上执行kubectl create创建calico服务

```bash
kubectl create -f calico.yaml 

# 返回结果如下
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
poddisruptionbudget.policy/calico-kube-controllers created
```

### 查看Calico状态

- 查看calico服务状态，待服务STATUS状态都为Running，说明服务启动成功

```bash
kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-67d4db6d8c-wfvz6   1/1     Running   0          2m
calico-node-2t7wr                          1/1     Running   0          2m
calico-node-nd92r                          1/1     Running   0          2m
calico-node-pnmpg                          1/1     Running   0          2m
calico-node-qsjms                          1/1     Running   0          2m
calico-node-t5gpx                          1/1     Running   0          2m
```

## 安装CoreDNS

### 编辑coredns.yaml文件

- 本文件安装的coredns为1.7.0版本，安装最新版，请按照下方**最新版calico安装**进行安装

```bash
cat > /tmp/coredns.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: coredns
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - services
  - pods
  - namespaces
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf {
          max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/name: "CoreDNS"
spec:
  # replicas: not specified here:
  # 1. Default is 1.
  # 2. Will be tuned in real time if DNS horizontal auto-scaling is turned on.
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      k8s-app: kube-dns
  template:
    metadata:
      labels:
        k8s-app: kube-dns
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: coredns
      tolerations:
        - key: "CriticalAddonsOnly"
          operator: "Exists"
      nodeSelector:
        kubernetes.io/os: linux
      affinity:
         podAntiAffinity:
           preferredDuringSchedulingIgnoredDuringExecution:
           - weight: 100
             podAffinityTerm:
               labelSelector:
                 matchExpressions:
                   - key: k8s-app
                     operator: In
                     values: ["kube-dns"]
               topologyKey: kubernetes.io/hostname
      imagePullSecrets:
      - name: my-secret
      containers:
      - name: coredns
        image: repository.test.com:8444/kubernetes/coredns/coredns:1.7.0
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        args: [ "-conf", "/etc/coredns/Corefile" ]
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
          readOnly: true
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - all
          readOnlyRootFilesystem: true
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 8181
            scheme: HTTP
      dnsPolicy: Default
      volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
---
apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  annotations:
    prometheus.io/port: "9153"
    prometheus.io/scrape: "true"
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "CoreDNS"
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 172.168.0.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
  - name: metrics
    port: 9153
    protocol: TCP
EOF
```

### 启动coredns服务

```bash
kubectl create -f /tmp/coredns.yaml

# 返回结果
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
```

### 查看coredns服务状态

```bash
kubectl get pod -n kube-system | grep coredns
coredns-55c474df8b-4xh6g                   1/1     Running   0          21s
```

### 最新版calico安装

```bash
git clone https://github.com/coredns/deployment.git
cd deployment/kubernetes
./deploy.sh -s -i 192.168.0.10 | kubectl apply -f -

# 返回结果
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

# 安装Metrics服务

官方yaml文件下载地址：https://github.com/kubernetes-sigs/metrics-server/releases

- Metrics服务用作采集kubernetes集群中的系统资源，比如：pod的内存、磁盘、CPU和网络的使用率

## 编辑metrics-server.yaml文件

```bash
cat > /tmp/metrics-server.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
  - apiGroups:
      - metrics.k8s.io
    resources:
      - pods
      - nodes
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - nodes
      - nodes/stats
      - namespaces
      - configmaps
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      imagePullSecrets:
      - name: my-secret
      containers:
        - args:
            - --cert-dir=/tmp
            - --secure-port=4443
            - --metric-resolution=30s
            - --kubelet-insecure-tls
            - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
            - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem # change to front-proxy-ca.crt for kubeadm
            - --requestheader-username-headers=X-Remote-User
            - --requestheader-group-headers=X-Remote-Group
            - --requestheader-extra-headers-prefix=X-Remote-Extra-
          image: repository.test.com:8444/kubernetes/metrics/metrics-server:v0.4.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /livez
              port: https
              scheme: HTTPS
            periodSeconds: 10
          name: metrics-server
          ports:
            - containerPort: 4443
              name: https
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /readyz
              port: https
              scheme: HTTPS
            periodSeconds: 10
          securityContext:
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
            - mountPath: /tmp
              name: tmp-dir
            - name: ca-ssl
              mountPath: /etc/kubernetes/pki
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
        - emptyDir: {}
          name: tmp-dir
        - name: ca-ssl
          hostPath:
            path: /etc/kubernetes/pki
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
EOF
```

## 启动metrics-server服务

```bash
kubectl create -f /tmp/metrics-server.yaml

# 返回结果
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

## 查看metrics服务状态

```bash
kubectl get pod -n kube-system | grep metrics
metrics-server-6f475c76f8-ggpcp            1/1     Running   0          2m21s
```

## 查看node节点状态

```bash
kubectl top node
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
10.202.43.114   230m         6%     841Mi           12%       
10.202.43.191   213m         6%     901Mi           13%       
10.202.43.46    386m         11%    2089Mi          31%       
10.202.43.55    525m         15%    1419Mi          21%       
10.202.43.56    358m         10%    1303Mi          19%
```

# 测试k8s集群是否可用

- 编辑busybox文件

```bash
cat > /tmp/busybox.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  imagePullSecrets:
  - name: my-secret
  containers:
  - name: busybox
    image: repository.test.com:8444/kubernetes/busybox/busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF
```

## 用bosybox解析默认命名空间中的kubernetes.svc

```bash
kubectl exec busybox -n default -- nslookup kubernetes

# 返回结果
Server:    172.168.0.2
Address 1: 172.168.0.2 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 172.168.0.1 kubernetes.default.svc.cluster.local
```

## 用bosybox测试跨命名空间是否可以解析

```bash
kubectl exec busybox -n default -- nslookup kube-dns.kube-system

# 返回结果
Server:    172.168.0.2
Address 1: 172.168.0.2 kube-dns.kube-system.svc.cluster.local

Name:      kube-dns.kube-system
Address 1: 172.168.0.2 kube-dns.kube-system.svc.cluster.local
```

## 每个节点都要能访问kubernetes的443和coredns的53

```bash
[root@k8s-master-2 tmp]# telnet 172.168.0.1 443
Trying 172.168.0.1...
Connected to 172.168.0.1.
Escape character is '^]'.
^CConnection closed by foreign host.
[root@k8s-master-2 tmp]# telnet 172.168.0.2 53
Trying 172.168.0.2...
Connected to 172.168.0.2.
Escape character is '^]'.
```

## pod和pod之间要能通信

```bash
[root@k8s-master-1 tmp]# kubectl get pod -o wide  -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE   IP                NODE            NOMINATED NODE   READINESS GATES
default       busybox                                    1/1     Running   0          18m   192.168.158.132   10.202.43.56    <none>           <none>
kube-system   calico-kube-controllers-67d4db6d8c-wfvz6   1/1     Running   0          46h   192.168.64.65     10.202.43.191   <none>           <none>
kube-system   calico-node-2t7wr                          1/1     Running   0          46h   10.202.43.56      10.202.43.56    <none>           <none>
kube-system   calico-node-nd92r                          1/1     Running   0          46h   10.202.43.191     10.202.43.191   <none>           <none>
kube-system   calico-node-pnmpg                          1/1     Running   0          46h   10.202.43.46      10.202.43.46    <none>           <none>
kube-system   calico-node-qsjms                          1/1     Running   0          46h   10.202.43.114     10.202.43.114   <none>           <none>
kube-system   calico-node-t5gpx                          1/1     Running   0          46h   10.202.43.55      10.202.43.55    <none>           <none>
kube-system   coredns-55c474df8b-4xh6g                   1/1     Running   0          71m   192.168.41.196    10.202.43.55    <none>           <none>
kube-system   metrics-server-6f475c76f8-ggpcp            1/1     Running   0          36m   192.168.242.194   10.202.43.46    <none>           <none>
[root@k8s-master-1 tmp]# kubectl exec -it busybox -- sh   # 进入busybox pod中
/ # ping 192.168.64.65   # ping其他pod测试
PING 192.168.64.65 (192.168.64.65): 56 data bytes
64 bytes from 192.168.64.65: seq=0 ttl=62 time=0.892 ms
64 bytes from 192.168.64.65: seq=1 ttl=62 time=0.973 ms
^C
--- 192.168.64.65 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.892/0.932/0.973 ms
/ # ping 10.202.43.55   # ping其他node节点测试
PING 10.202.43.55 (10.202.43.55): 56 data bytes
64 bytes from 10.202.43.55: seq=0 ttl=63 time=0.561 ms
^C
--- 10.202.43.55 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.561/0.561/0.561 ms
```

