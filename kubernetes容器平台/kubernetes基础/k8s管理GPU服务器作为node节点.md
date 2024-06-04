### 1.确定显卡型号并下载驱动

确定显卡型号

```bash
➜  ~ lspci | grep NV
01:00.0 VGA compatible controller: NVIDIA Corporation GM200 [GeForce GTX TITAN X] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GM200 High Definition Audio (rev a1)
02:00.0 VGA compatible controller: NVIDIA Corporation GM200 [GeForce GTX TITAN X] (rev a1)
02:00.1 Audio device: NVIDIA Corporation GM200 High Definition Audio (rev a1)
```

下载对应型号的显卡驱动：https://www.nvidia.cn/Download/index.aspx?lang=cn

### 2.升级内核并且安装驱动

**升级内核**

内核升级必须安装以下三个包且版本必须一致，`kernel-lt-headers`包安装可能报错需执行`yum remove kernel-headers`卸载原始安装包之后安装。

内核rpm下载地址：https://mirrors.tuna.tsinghua.edu.cn/elrepo/

```bash
yum install kernel-lt-5.4.210
yum install kernel-lt-devel-5.4.210
yum install kernel-lt-headers-5.4.210
#切换内核
cp /boot/grub2/grub.cfg /boot/grub2/grub.cfg.bak
grub2-set-default 0
grub2-mkconfig -o /boot/grub2/grub.cfg
#重启
reboot
```

**升级gcc9**

需连接公网，如果连接不了公网需要自己同步rpm包并制作离线仓库。

```bash
sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/centos-release-scl-rh-2-3.el7.centos.noarch.rpm
sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/centos-release-scl-2-3.el7.centos.noarch.rpm
sudo yum install devtoolset-9-gcc-c++
#临时启用，安装驱动时需要启用
source /opt/rh/devtoolset-9/enable
#验证
g++ --version
```

**编译安装驱动**

```bash
chmod +x NVIDIA-Linux-x86_64-515.65.01.run

#全部选yes
./NVIDIA-Linux-x86_64-515.65.01.run 

#测试，正常输出显卡信息表示正常
nvidia-smi 
```

### 3.安装nvidia-docker

**首先需要安装docker**

可以配置docker-ce源进行rpm安装。

```bash
#配置源
vim /etc/yum.repo.d/docker-ce.repo
[docker-ce]
name=docker-ce
baseurl=https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/x86_64/stable/
gpgcheck=0
#安装，可自己选择版本推荐18.06版本及以上nvidia-docker2依赖
yum install docker-ce-19.03.15
#启动
systemctl restart docker
```

**部署nvidia-docker**

这里推荐安装nvidia-docker2

官方配置说明：https://nvidia.github.io/libnvidia-container/

```bash
#配置源
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.repo | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
#安装
sudo yum install -y nvidia-docker2
```

**修改docker默认运行时**

```bash
cat /etc/docker/daemon.json 
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
#重启服务
systemctl restart docker
#验证
docker info
#验证docker是否可以正常使用显卡，有显卡型号输出表示正常
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

### 4.部署k8s组件

具体组件部署略，下面补充一些kubelet使用docker配置，其余与其他node节点基本一致。

```bash
  --container-runtime=docker \
  --container-runtime-endpoint=unix:///var/run/dockershim.sock \
  --pod-infra-container-image=law-harbor.internal.gridsumdissector.com/demo/pause:3.1 \
```

最好给节点打标签以及污点

```bash
kubectl label node 10.136.88.43 type=gpu
kubectl taint node 10.136.88.43 type=gpu:NoExecute
```

### 5.部署k8s设备插件

官方文档：https://kubernetes.io/zh-cn/docs/tasks/manage-gpus/scheduling-gpus/

```
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml
```

### 6.验证

运行以下pod，查看日志确定容器中是否识别显卡

```bash
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  tolerations:  #注意如果打了污点，这里必须匹配污点
  - key: "type"
    operator: "Exists"
    effect: "NoExecute"
  nodeSelector:  #选择指定gpu节点调度
    type: "gpu"
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      # https://github.com/kubernetes/kubernetes/blob/v1.7.11/test/images/nvidia-cuda/Dockerfile
      image: "law-harbor.internal.gridsumdissector.com/demo/cuda:11.0.3-base-ubuntu20.04"
      command:
      - nvidia-smi
      resources:  #注意显卡的资源限制必须为整数，且不可以设置请求资源，如果不写gpu资源限制即表示不限制
        limits:
          nvidia.com/gpu: 2
```

