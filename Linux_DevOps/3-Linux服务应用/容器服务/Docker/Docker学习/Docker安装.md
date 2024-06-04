## CentOS 系统安装 Docker

Docker 支持CentOS 6 及以后的版本

### CentOS 6.x 安装 Docker

对于 CentOS 6.X ，可以使用 EPEL（https://fedoraproject.org/wiki/EPEL） 库安装 Docker ，命令如下：

```shell
$ sudo yum install http://mirrors.yun-idc.com/epel/6/i386/epel-release-6-8.noarch.rpm
$ sudo yum install docker-io
```

### CentOS 7.x 安装 Docker

CentOS 7.x 系统自带的 CentOS-Extras 库中已带 Docker ，可以直接安装

```shell
$ sudo yum install -y docker
```

### 启动 Docker

安装之后启动 docker 服务，并让它随系统启动自动加载

```shell
CentOS 6.x
$ service docker start
$ chkconfig docker on

CentOS 7.x
$ systemctl start docker
$ systemctl enable docker
```

## Ubuntu 系统安装 Docker

### 通过系统自带包安装

Ubuntu 14.04 版本，系统中已经自带了 Docker 包，可以直接安装。

```shell
$ sudo apt-get update
$ sudo apt-get install -y docker.io
$ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
$ sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io
```

如果使用操作系统自带包安装 Docker，目前安装的版本是比较旧的 0.9.1 。 要安装更新的版本，可以通过使用 Docker 源的方式。

### 通过 Docker 源安装最新版本

要安装最新的 Docker 版本，首先需要安装 apt-transport-https 支持，之后通过添加源来安装。

```shell
$ sudo apt-get install apt-transport-https
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
$ sudo bash -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
$ sudo apt-get update
$ sudo apt-get install lxc-docker
```

### 14.04 之前版本安装

如果是较低版本的Ubuntu系统，需要先升级内核。

```shell
$ sudo apt-get update
$ sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring
$ sudo reboot
```

然后重启生面的步骤即可

### 启动 Docker 服务

```shell
$ sudo service docker start
```