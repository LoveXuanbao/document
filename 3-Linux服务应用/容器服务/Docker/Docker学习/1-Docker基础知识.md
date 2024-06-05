https://wiki.jikexueyuan.com/project/docker-technology-and-combat/why.html



# 一、什么是Docker

`Docker`是一个开源项目，诞生于 2013 年初，最初是 `docCloud` 公司内部的一个业余项目。他它基于 Google 公司推出的 Go 语言实现。项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 [GitHub](https://github.com/moby/moby)上进行维护。

Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。Redhat 已经在其 RHEL6.5 中集中支持 Docker；Google 也在其 PaaS 产品中广泛应用。

Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。

在 LXC（Linux原生支持的容器技术） 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口(类似 iphone 的 APP)。几乎没有性能开销，可以很容易的在机器和数据中心运行。最重要的是，他们不依赖于任何语言、框架包括系统

Docker是和伟大的项目，它彻底释放了虚拟化的威力，极大降低了云计算资源供应的成本，同时让应用的分发、测试、部署都变得前所未有的高效和轻松！

# Docker 与 VM 的区别

![image-20230825150246954](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825150246954.png)

## Docker 与 openstack 的对比

| 类别     | Docker                | OpenStack                           |
| -------- | --------------------- | ----------------------------------- |
| 部署难度 | 非常简单              | 组件多，部署负责                    |
| 启动速度 | 秒级                  | 分钟级                              |
| 执行性能 | 和物理系统几乎一致    | VM会占用一些资源                    |
| 镜像体积 | 镜像是MB级别          | 虚拟机镜像GB级别                    |
| 管理效率 | 管理简单              | 组件相互依赖，管理复杂              |
| 隔离性   | 隔离性高              | 彻底隔离                            |
| 可管理性 | 单进程，不建议启动SSH | 完整的系统管理                      |
| 网络连接 | 比较弱                | 借助Neutron可以灵活组件各类网络架构 |




# 为什么要使用 Docker

作为一种新兴的虚拟化方式，Docker 跟传统的虚拟化方式相比具有众多的优势。

首先，Docker 容器的启动可以在秒级实现，这相比传统的虚拟机方式要快得多。其次，Docker 对系统资源的利用率很高。一台主机上可以同时运行数千个 Docker 容器。

容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而 Docker 只需要启动 10 个隔离的应用即可。

具体来说，Docker 在如下几个方面具有较打的优势。

## 更快速的交付和部署

对开发和运维（devop）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。

开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。

## 让开发和拓展更加简单

Docker 以容器为基础的平台运行高度移植的工作。Docker 容器可以在开发者机器上运行，也可以在实体或者虚拟机上运行，也可以在云平台上运行。Docker的可移植、轻量特性同样让动态地管理负载更加简单。你可以用 Docker 快速地增加应用规模或者关闭应用程序和服务。Docker 的快速意味着变动几乎是实时的。

## 更高效的虚拟化

Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。

Docker 轻巧快速，它提供了一个可行的、符合成本效益的替代基于虚拟机管理程序的虚拟机。这在高密度的环境下使用尤其有用。例如：构建你自己的云平台或者PaaS，在中小的部署环境下童谣可以获取到更多的资源性能。

## 更轻松的迁移和扩展

Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。

## 更简单的管理

使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。

对比传统虚拟机总结

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 磁盘使用   | 一般为MB           | 一般为GB   |
| 性能       | 接近原生           | 弱于原生   |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |
# 基本概念

`Docker`包括三个基本概念

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

理解了这三个概念，就理解了`Docker`的整个生命周期。

## 1. Docker 镜像

Docker 镜像就是一个只读的模板。每一个镜像由一系列的层（layers）组成。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独立文件系统中的文件和文件夹（称之为分支）被透明覆盖，行程一个段都连贯的文件系统。正因为有了这些层的存在，Docker是如此的轻量。当你改变了一个 Docker 镜像。比如升级某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的奖项或者重新建立（在使用虚拟机的时候可能会这么做），只是一个新的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

例如：一个镜像可以包含一个完整的 CentOS 操作系统环境，里面仅安装了 Apache 或用户需要的其他应用程序

镜像可以用来创建 Docker 容器

Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用

#### 镜像的实现原理

- Dokcer镜像时怎么实现增量的修改和维护的？
- 每个镜像都是有很多层次构成，Docker使用`Union FS`将这些不同的层结合到一个镜像中去。
- 通常`Union FS`有两个用途，一方面可以实现不借助`LVM`、`RAID`将多个`disk`（磁盘）挂载到同一个目录下，另一个更常用的就是将一个只读的分区和一个可写的分支联合在一起，`Live CD`正式基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作。Docker在AUFS上构建的容器也是利用了类似的原理。

#### 获取/拉取镜像

- 镜像是Docker的三大组件之一。Docker运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker会从镜像仓库下载（默认是Docker Hub公共注册服务器中的仓库）

- 可以使用`docker pull`命令来从仓库获取所需要的镜像。

  例如：从阿里云仓库下载一个centos的镜像

```bash
[root@localhost docker]# docker pull centos
Using default tag: latest
Trying to pull repository docker.io/library/centos ... 
latest: Pulling from docker.io/library/centos
8a29a15cefae: Pull complete 
Digest: sha256:fe8d824220415eed5477b63addf40fb06c3b049404242b31982106ac204f6700
Status: Downloaded newer image for docker.io/centos:latest
[root@localhost docker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              470671670cac        2 months ago        237 MB
```

- 下载过程中，会输入获取镜像的每一层信息。
- 完成后，即可随时使用该镜像，例如创建一个容器，让其中运行bash应用

```bash
[root@localhost docker]# docker run -t -i centos /bin/bash
[root@316643beb772 /]#
```

#### 列出镜像

- 使用`docker images`显示本地已有的镜像

```bash
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx      latest              6678c7c2e56c        2 weeks ago         127 MB
docker.io/registry   latest              708bc6af7e5e        2 months ago        25.8 MB
docker.io/centos     latest              470671670cac        2 months ago        237 MB
```

- 在列出的信息中，可以看到几个字段信息
  - REPOSITORY：代表来自哪个仓库，比如`docker.io/centos`
  - TAG：镜像的标记和版本，如果`latest`
  - IMAGE ID：镜像的ID号（唯一）
  - CREATED：镜像创建的时间
  - SIZE：镜像的大小
- 其中镜像的ID唯一标识了镜像，注意到如果具有相同的镜像ID，说明他们实际上是同一个镜像
- TAG信息用来标记来自同一个仓库的不同镜像，例如`docker.io/centos`仓库中有多个镜像，通过TAG信息来区分发行版本，例如`7.1`、`7.2`、`7.3`等等。如果不指定具体的标记，则默认使用`latest`标记信息，例如下面的命令指定使用镜像`docker.io/centos:latest`来启动一个容器

```bash
[root@localhost ~]# docker run -t -i docker.io/centos:latest /bin/bash
[root@62d0aef6825b /]#
```

#### 创建镜像

- 创建镜像有很多方法，用户可以从Docker Hub获取已有镜像并更新，也可以利用本地文件系统创建一个。

#### 修改已有镜像

- 创建一个容器`mynginx`，使用`centos`镜像

```bash
[root@localhost ~]# docker run --name mynginx -it centos
[root@ba019ebb2cfa /]# rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
Retrieving http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
warning: /var/tmp/rpm-tmp.j1VKH1: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:epel-release-7-12                ################################# [100%]
[root@ba019ebb2cfa /]# yum install -y nginx 
安装过程不在显示
[root@ba019ebb2cfa /]# exit
exit
```

- 基于`mynginx`容器做一个镜像`mynginx:v1`

```bash
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
ba019ebb2cfa        centos              "/bin/bash"         5 minutes ago       Exited (130) 54 seconds ago                       mynginx

基于 mynginx 这个容器做一个镜像
[root@localhost ~]# docker commit -m "my nginx" ba019ebb2cfa test/mynginx:v1
sha256:2c5933400155b0272aaf99b6cbf5758717027abbd3776c2e38b204181d52f25f
提交镜像，同时打一个标签叫: mynginx:v1，test相当于你向github上提交的用户名

查看镜像
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
test/mynginx         v1                  2c5933400155        17 seconds ago      356 MB
docker.io/nginx      latest              6678c7c2e56c        2 weeks ago         127 MB
docker.io/centos     latest              470671670cac        2 months ago        237 MB
```

- 基于`mynginx:v1`镜像创建一个容器`mynginxv1`，目的是修改`nginx`不让其在后台运行

```bash
[root@localhost ~]# docker run -it --name mynginxv1 test/mynginx:v1
[root@a6eb174dbd23 /]# vi /etc/nginx/nginx.conf
daemon off;     ##不在后台运行

[root@a6eb174dbd23 /]# exit
exit
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
a6eb174dbd23        test/mynginx:v1     "/bin/bash"         2 minutes ago       Exited (0) 19 seconds ago                        mynginxv1
```

- 基于`mynginxv1`提交`mynginxv2`版本

  - 重新提交v2版本

  ```bash
  [root@localhost ~]# docker commit -m "my nginx" a6eb174dbd23 test/mynginx:v2
  sha256:2d96d85f2f1ba902fbea2e307f563ab7a38df2aa8c6f909ccd3fa470f5539f22
  
  查看 v2 镜像
  [root@localhost ~]# docker images
  REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
  test/mynginx         v2                  2d96d85f2f1b        3 seconds ago       356 MB
  test/mynginx         v1                  2c5933400155        9 minutes ago       356 MB
  ```

- 基于`mynginxv2`镜像，创建`mynginxv2`容器

  - 启动容器：`-d`后台运行，`-p`指定端口，在后面是经i想，最后是命令（因为是yum安装的，可以直接写nginx，如果不是yum，需要些绝对路径）

    ```bash
    [root@localhost ~]# docker run -d -p 8888:80 test/mynginx:v2 nginx
    09667e02a9c9e63f17fb27b5fa185f5993056a56b7fb90390a63a37431113b3a
    
    [root@localhost ~]# docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS                  NAMES
    09667e02a9c9        test/mynginx:v2     "nginx"             4 seconds ago       Up 3 seconds                  0.0.0.0:8888->80/tcp   dazzling_wiles
    ```

  - 可以在浏览器访问8888端口

  ![image-20230825153047603](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825153047603.png)

#### 修改镜像TAG

```bash
# 查看镜像
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
test/mynginx         v2                  2d96d85f2f1b        3 seconds ago       356 MB
test/mynginx         v1                  2c5933400155        9 minutes ago       356 MB

# 修改镜像tag
[root@localhost ~]# docker tag test/mynginx:v2 test/mynginx-change:v2-change

# 再次查看
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
test/mynginx-change  v2-change           2d96d85f2f1b        3 seconds ago       356 MB
test/mynginx         v2                  2d96d85f2f1b        3 seconds ago       356 MB
test/mynginx         v1                  2c5933400155        9 minutes ago       356 MB

```



#### 利用Dockerfile来创建镜像

- 使用`docker commit`来扩展一个镜像比较简单，但是不方便在一个团队中分享。我们可以使用`docker build`来创建一个新的镜像。为此，首先需要创建一个`Dockerfile`文件，文件中包含如何创建镜像的指令。

##### Dockerfile包含的信息

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行的指令

##### Dockerfile文件的编写

```bash
[root@localhost ~]# mkdir /opt/dockerfile/nginx/ -p
[root@localhost ~]# cd /opt/dockerfile/nginx/

创建一个index.html文件
[root@localhost nginx]# vim index.html 
hello world!

[root@localhost nginx]# vim Dockerfile 
# This is docker file
# version v1
# Author wangshibo
# Base image(基础镜像)
FROM centos
 
# Maintainer(维护者信息)
MAINTAINER niuzhan  956391432@qq.com
 
# Commands(执行命令)
RUN rpm -ivh  http://mirrors.aliyun.com/epel/epel-release-latest-8.noarch.rpm
RUN yum -y install nginx
# Add(添加文件)
# index.html是自己编写的文件，放在后面的目录中，因为yum安装后Documentroot是在这里
ADD index.html /usr/share/nginx/html/index.html    

RUN echo "daemon off;" >>/etc/nginx/nginx.conf

# 对外的端口
EXPOSE 80           

# 执行的命令
CMD ["nginx"]
```

| FROM     | MAINTAINER | RUN      | ADD                  | WORKDIR         | VOLUME   | EXPOT        | CMD                    | ENV          |
| -------- | ---------- | -------- | -------------------- | --------------- | -------- | ------------ | ---------------------- | ------------ |
| 基础镜像 | 维护者信息 | 安装软件 | COPY文件，会自动解压 | cd 切换工作目录 | 目录挂载 | 内部服务端口 | 执行Dockerfile中的命令 | 设置环境变量 |

- 编写完成Dockerfile后使用`docker build`来生成镜像

```bash
[root@localhost nginx]# docker build -t test/mynginx:v1 ./
Sending build context to Docker daemon 3.584 kB
Step 1/8 : FROM centos
 ---> 470671670cac
Step 2/8 : MAINTAINER niuzhan  956391432@qq.com
 ---> Using cache
 ---> 9b59ce9b169e
Step 3/8 : RUN rpm -ivh  http://mirrors.aliyun.com/epel/epel-release-latest-8.noarch.rpm
 ---> Using cache
 ---> 9cf07f7bf5aa
Step 4/8 : RUN yum -y install nginx
 ---> Running in 6c3e5011eef0
具体安装步骤已省略
Complete!
 ---> 1c4c35d41df0
Removing intermediate container 6c3e5011eef0
Step 5/8 : ADD index.html /usr/share/nginx/html/index.html
 ---> c8945d2726f0
Removing intermediate container 0e9483df5ace
Step 6/8 : RUN echo "daemon off;" >>/etc/nginx/nginx.conf
 ---> Running in 8dc5ed20bfa0

 ---> a68ef39928a0
Removing intermediate container 8dc5ed20bfa0
Step 7/8 : EXPOSE 80
 ---> Running in cb44bdb165c2
 ---> f4441a885cff
Removing intermediate container cb44bdb165c2
Step 8/8 : CMD nginx
 ---> Running in 9c4b68f25a04
 ---> 7cd9e43840da
Removing intermediate container 9c4b68f25a04
Successfully built 7cd9e43840da
```

- 其中`-t`标记来添加tag，指定新得镜像得用户细信息

- "./" 是Dockerfile所在得路径（当前目录），也可以替换为一个具体得Dockerfile得绝对路径

- 可以看到build进程在执行得操作，它做得第一件事情就是上传这个Dockerfile内容，因为所有得操作都要依据Dockerfile中得指令被一条一条得执行。没一部都创建一个新得容器，在容器中执行指令并提交修改（和docker commit一样）。当所有得指令都执行完毕之后，返回了最终得镜像id。所有得中间步骤所产生得容器都被删除和清理了。

  **注意** ：一个镜像不能超过127层。

#### 运行新的镜像构建容器

```bash
[root@localhost nginx]# docker run -d -p 8888:80 test/mynginx:v1
4f6d8337d8023063fb62d43fd028412dc6066250a4099c75aebd06e7a20ce55a
[root@localhost nginx]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                  NAMES
4f6d8337d802        test/mynginx:v1     "nginx"                  4 seconds ago       Up 4 seconds               0.0.0.0:8888->80/tcp   confident_leakey
```

#### 访问

- 浏览器输入IP和端口来访问

![image-20230825153133848](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825153133848.png)

#### 从本地文件系统导入镜像

- 要从本地文件系统导入一个镜像，可以使用openvz（容器虚拟化得先锋技术）得模板来创建：openvz得模板下载地址为：templates（https://wiki.openvz.org/Download/template/precreated）
- 比如，先下载一个centos7.4.1708.tar.gz的镜像的时候使用以下命令导入
  - docker load -i imagename
  - docker load < imagename
  - docker import - imagename

```bash
[root@localhost ~]# docker load -i centos7.4.1708.tar.gz 
606d67d8e1b8: Loading layer [==================================================>] 204.7 MB/204.7 MB
Loaded image: docker.io/centos:centos7.4.1708

然后查看导入的镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    centos7.4.1708      9f266d35e02c        12 months ago       197 MB
```



#### 上传镜像

- 用户可以通过 docker push 命令，把自己创建的镜像上传导仓库中来共享。例如：用户在 docker hub 上完成注册后，可以推送到自己的镜像到仓库中，或者推送到自己搭建的私有仓库中。
- 步骤说明：
  - **docker tag**    先把原有的镜像重新打个标签，打成自己私有仓库的地址标签；
  - **docker login**  登录自己的 nexus 私有仓库；
  - **docker push**  上传镜像；

```bash
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    centos7.4.1708      9f266d35e02c        12 months ago       197 MB
[root@localhost ~]# docker tag docker.io/centos:centos7.4.1708 repository.test.com:8444/libary/centos:v7.4.1708
[root@localhost ~]# docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos                         centos7.4.1708      9f266d35e02c        12 months ago       197 MB
repository.test.com:8444/libary/centos   v7.4.1708           9f266d35e02c        12 months ago       197 MB
[root@localhost ~]# docker login repository.test.com:8444
Username: admin
Password: 
Login Succeeded
[root@localhost ~]# docker push repository.test.com:8444/libary/centos:v7.4.1708
The push refers to a repository [repository.test.com:8444/libary/centos]
606d67d8e1b8: Pushed 
v7.4.1708: digest: sha256:78b63c089c40a2ca0d85db9125fb3193c6ff74814bba5f894c803e5ae02a4035 size: 529
```



#### 导出和导入镜像

##### 导出镜像

- 如果要导出镜像到本地文件系统中，可以使用 **docker save**命令；例如：

- **docker save -o centos7.4.1708.tar.gz repository.test.com:8444/libary/centos:v7.4.1708**    也可以写成  **docker save repository.test.com:8444/libary/centos:v7.4.1708 -o centos7.4.1708.tar.gz**  效果是一样的。

```bash
[root@localhost docker-images]# docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos                         centos7.4.1708      9f266d35e02c        12 months ago       197 MB
repository.test.com:8444/libary/centos   v7.4.1708           9f266d35e02c        12 months ago       197 MB
[root@localhost docker-images]# docker save -o centos7.4.1708.tar.gz repository.test.com:8444/libary/centos:v7.4.1708
[root@localhost docker-images]# ll
总用量 199940
-rw------- 1 root root 204738560 3月  26 11:49 centos7.4.1708.tar.gz
```

##### 导入镜像

- 与本地文件系统导入镜像一致
- 使用 **docker load**从导出或下载的本地文件系统中的镜像导入到本地镜像库，例如：

```bash
docker load --input centos7.4.1708.tar.gz  &&  docker load -i centos7.4.1708.tar.gz
或
docker load < centos7.4.1708.tar.gzz
在或者
cat centos7.4.1708.tar.gz | docker import - centos7.4.1708.tar.gz
```



#### 删除镜像

- 如果要移除本地的镜像，可以使用 **docker rmi**命令，
- **注意： **在删除镜像之前要先用 docker rm 删除掉依赖于这个镜像的容器

```bash
[root@localhost docker-images]# docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
repository.test.com:8444/libary/centos   v7.4.1708           9f266d35e02c        12 months ago       197 MB
[root@localhost docker-images]# docker rmi repository.test.com:8444/libary/centos:v7.4.1708
Untagged: repository.test.com:8444/libary/centos:v7.4.1708
Deleted: sha256:9f266d35e02cc56fe11a70ecdbe918ea091d828736521c91dda4cc0c287856a9
Deleted: sha256:606d67d8e1b88099cf33bcbe675d0fb4f9035432f540a0d8be46017303c5cbe7
```



## 2. Docker 容器

- 容器是Docker的另一个核心概念。容器和文件夹类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个Docker容器都是从Docker镜像创建的，Docker 利用容器来运行应用。容器是从镜像创建的运行实例。Docker容器可以运行、开始、停止、移动和删除。每一个Docker容器都是独立和安全的应用平台，Docker容器时Docker的运行部分。
- 简单的说，容器时独立运行的一个或一组应用，以及它们的运行态环境。对应的虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用

- 可以把容器看作是一个简易版的 Linux 环境（包括 root 用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

注：镜像是只读的，容器在启动的创建一层可写层作为最上层

- 下面记录下Docker容器的底层技术：Namespace（资源隔离）和Cgroup（资源限制）

#### 命名空间【Namespace】

- pid namespace：使用在进程隔离（Process ID）
  - 不同用户的进程就是通过`pid namespace`隔离开的，且不同`namespace`中可以有相同的`PID`。
  - 具体特征如下：
    1. 每个`namespace`中的`pid`时有自己的`pid = 1`的进程（类似/sbin/init进程）；
    2. 每个`namespace`中的进程只能影响自己的同一个`namespace`或`子namespace`中的进程；
    3. 因为`/proc`包含正在运行的进程，因此在`container`中的`psedo-filesystem`的`/proc`目录只能看到自己的`namespace`中的进程；
    4. 因为`namespace`允许嵌套，`父namespace`可以影响`子namespace`的进程，所以`子namespace`的进程可以在`父namespace`中看到，但是具有不同的`pid`
- mnt namespace：使用在管理挂载点（Mount）
  - 类似`chroot`，将一个进程放到一个特定的目录执行，`mnt namespace`允许不同`namespace`的进程看到的文件结构不同，这样每个`namespace`中的进程所看到的文件目录就被隔离开了。同`chroot`不同，每个`namespace`中的`container`在`/proc/mounts`的信息只包含所在的`namespace`的`mount point`
- net namespace：使用在进程网络接口（NetWorking）
  - 网络隔离是通过`net namespace`实现的，每个`net namespace`有独立的`network devices`、`IP address`、`IP routing tables`、`/proc/net`目录。这样每个`container`的网络就能隔离开来。Docker默认采用`veth`的方式将`container`中的虚拟网卡同`host`上的一个`docker bridge`连接在一起
- uts namespace：使用在隔离内核和版本表示（Unix Timesharing System）
  - UTS（Unix Time-sharing System）namespace允许每个`container`拥有独立的`hostname`和`damain name`，使其在网络上可以被视作一个独立的节点而非`host`上的一个进程
- ipc namespace：使用在管理进程间通信资源（InterProcess Communication）
  - `container`中进程交互还是采用Linux常见的进程间交互方法（InterProcess Communication - IPC），包括常见的信号量、消息队列和共享内存。然后同VM不同，`container`的进程间交互实际上还是`host`上具有相同`pid namespace`中的进程间交互。因此需要在IPC资源折你去哪个时加入`namespace`信息，每个IPC资源有一个唯一的`32bit ID`
- user namespace：使用在管理用户空间
  - 每个`container`可以有不同的`user`和`group id`，也就时说可以以`container`内部的用户在`container`内部执行程序而非`host`上的用户

> 有了以上六种`namespace`从进程、网络、IPC、文件系统、UTS和用户角度的隔离，一个`container`就可以对外展现出一个独立计算机的能力，并且不同`container`从OS层面实现了隔离。然而不同`namespace`之间资源还是相互竞争的，仍然需要类似`ulimit`来管理每个`container`所能使用的资源。

#### 资源配置【Cgroups】

- Docker还使用到了`cgroups`技术来管理群组。使应用管理运行的关键是让他们只使用你想要的资源。这样可以确保在机器上运行的容器都是良民（good multi-tenant citizens）。群组控制允许Docker分享或限制容器使用硬件资源。例如：限制指定的容器的内容使用
- Cgroups实现了对资源的配额和度量。`cgroups`的使用非常简单，提供了类似文件的接口，在`/cgroup`目录下新建一个文件夹即可新建一个`group`，在此文件夹中新建`task`文件，并将`pid`写入该文件，即可实现对该进程的资源控制。具体的资源配置选项可以在该文件夹中新建`子subsystem`，{子系统前缀}.{资源项}是典型的配置方法。如`memory.usageinbutes`就定义了`group`在`subsystem memory`中的一个内存限制选项。另外，cgroup中的`subsystem`可以随意组件，一个`subsystem`可以在不同的`group`中，也可以一个`group`中包含多个`subsystem`

- memory
  - 内存相关的限制
- cpu
  - 在cgroup中，并不能像硬件虚拟化方案一样能够定义CPU能力，但是能够定义CPU轮转的优先级，因此具有较高的CPU优先级的进程会更可能得到CPU运算。通过将参数写入`cpu.shares`，即可定义该`cgroup`的CPU优先级，这里是一个相对权重，而非绝对值
- blkio
  - `block IO`相关的统计和限制，`byte/operation`统计和限制（IOPS等），读写的速度限制等，但是这里主要统计的都是同步IO

#### 启动容器

- 启动容器都有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动
- 因为Docker的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器

##### 新建并启动

- 所需要的命令主要为：`docker run`
- 例如：下面的命令输入一个`"Hello world"`之后终止容器

```
[root@localhost docker-images]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    centos7.4.1708      9f266d35e02c        12 months ago       197 MB
[root@localhost docker-images]# docker run -i -t docker.io/centos:centos7.4.1708 /bin/echo "Hello World"
Hello World
```

- 这跟本地直接执行`/bin/echo "Hello world"`几乎感觉不出任何区别。
- 下面的命令则启动一个bash终端，允许用户进行交互

```
[root@localhost docker-images]# docker run -it docker.io/centos:centos7.4.1708 /bin/bash
[root@c96df6d43a96 /]#
```

> 其中 `-t`选项让docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上，`-i`则让容器的标准输入保持打开

- 在交互模式下，用户可以通过所创建的终端来输入命令，例如：

```
[root@c96df6d43a96 ~]# pwd
/
[root@c96df6d43a96 /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

- 当利用`docker run`来创建容器时，Docker在后台运行的标准操作包括：
  1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
  2. 利用镜像创建并启动一个容器；
  3. 分配一个文件系统，并在制度的镜像层外挂载一层可读可写；
  4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
  5. 从地址池配置一个IP地址给容器；
  6. 执行用户指定的应用程序；
  7. 执行完毕后容器被终止。

##### 启动已终止的容器

- 可以利用`docker start`命令，直接将一个已经终止的容器启动运行。
- 容器的核心为所执行的应用程序，所需要的资源都时应用程序运行所必须的。除此之外，并没有其他的资源。可以在伪终端中利用`ps`或`top`来查看进程信息。

```
[root@c96df6d43a96 /]# ps
   PID TTY          TIME CMD
     1 ?        00:00:00 bash
    20 ?        00:00:00 ps
[root@c96df6d43a96 /]# top
top - 07:56:50 up  5:23,  0 users,  load average: 0.10, 0.17, 0.21
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   999696 total,    64144 free,   744180 used,   191372 buff/cache
KiB Swap:  2097148 total,  1654232 free,   442916 used.    60136 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                          
     1 root      20   0   11752   1912   1524 S  0.0  0.2   0:00.05 bash        
    21 root      20   0   51924   1892   1396 R  0.0  0.2   0:00.00 top 
```

- 可见，容器中仅运行了指定的bash应用，这种特点使得Docker对资源的利用率极高，时货真价实的轻量级虚拟化。

##### 后台运行

- 更多的时候，需要让Docker容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加`-d`参数来实现

```
[root@localhost docker-images]# docker run -d centos:centos7.4.1708 /bin/sh -c "while true; do echo hello world; sleep 1; done"
624e1c74fa96e66fb55fdb7dd080c74045640f63db0c4b43e309b46fa5a9b747
[root@localhost docker-images]# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
624e1c74fa96        centos:centos7.4.1708   "/bin/sh -c 'while..."   2 seconds ago       Up 1 second                             gifted_keller
```

- 要获取容器的输出信息，可以通过`docker logs`命令

```
[root@localhost docker-images]# docker logs 624e1c74fa96
hello world
hello world
hello world
```

#### 终止容器

- 可以使用`docker stop`来终止一个运行中的容器
- 此外，当Docker容器中指定的应用终结时，容器也会自动终止。例如我们之前启动了终端的容器，可以通过`exit`命令或者`ctrl + d`来退出终端，所创建的容器即可终止。
- 终止状态的容器可以用`docker ps -a`命令看到，例如：

```
[root@localhost docker-images]# docker ps -a
CONTAINER ID        IMAGE                             COMMAND                  CREATED              STATUS                       PORTS               NAMES
624e1c74fa96        centos:centos7.4.1708             "/bin/sh -c 'while..."   About a minute ago   Exited (137) 2 seconds ago                       gifted_keller
ebdc635253cb        docker.io/centos:centos7.4.1708   "/bin/sh -c 'while..."   2 minutes ago        Exited (1) 2 minutes ago                         musing_jang
8c96f2c021ab        docker.io/centos:centos7.4.1708   "/bin/bash -c 'whi..."   2 minutes ago        Exited (1) 2 minutes ago   
```

- 处于终止状态的容器，可以通过`docker start ID`命令来重新启动，此外，`docker restart ID`命令会将一个运行中的容器终止，然后再重新启动它

#### 进入容器

- 在使用`-d`参数时，容器启动后会进入后台运行，某些时候需要进入容器进行操作，有很多中方法，包括使用`docker attach`、`ssh`、`docker exec `或`nsenter`工具等

##### 使用`attach`命令进入Docker容器

- docker attach是Docker自带的命令。如下：

```
[root@localhost docker-images]# docker run -itd docker.io/centos:centos7.4.1708
12c67284c170fd451a05ea96b9263450bccaf9e53c0a5824360f9d79c19f4353
[root@localhost docker-images]# docker ps
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS              PORTS               NAMES
12c67284c170        docker.io/centos:centos7.4.1708   "/bin/bash"         2 seconds ago       Up 1 second                             serene_liskov
[root@localhost docker-images]# docker attach serene_liskov
```

> 但是使用`attach`命令的时候并不方便，当多个窗口同时`attach`到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时，其他窗口也无法执行操作。因为这个原因，所以`docker attach`命令不太适合用于生产环境，平台自己开发测试应用时可以使用该命令

##### 使用SSH进入Docker容器

- 使用SSH进入Docker容器，需要在镜像（或容器）中安装`SSH Server`，这样就能够保证多人进入容器且相互之间不受干扰，但是使用Docker容器之后不建议使用ssh进入到Dokcer容器内，关于为什么不建议使用，请参考下面连接中的文件：https://www.oschina.net/translate/why-you-dont-need-to-run-sshd-in-docker?cmp

##### 使用`nsenter`命令进入Docker容器

- 关于什么时`nsenter`请参考如下文章：https://github.com/jpetazzo/nsenter
- 系统默认将我们需要的`nsenter`安装到主机中（`nsenter`工具在`util-linux.2.23`版本之后包含），如果没有安装的话，按照下面步骤从源码安装即可（注意是主机而非容器或镜像）

```
$ wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz  
$ tar -xzvf util-linux-2.24.tar.gz  
$ cd util-linux-2.24/  
$ ./configure --without-ncurses  
$ make nsenter  
$ sudo cp nsenter /usr/local/bin 
```

- `nsenter`可以访问另一个进程中的命名空间，`nsenter`要正常工作需要有`root`权限。所以我们为了连接到某个容器我们还需要获取该容器的第一个进程的`PID`，可以使用`docker inspect ID`命令来拿到该`PID`

- 由于该信息非常多，此处只截取了容器的一部分内容进行展示，如果要显示该容器第一个进程的`PID`可以使用如下方式：

```
[root@localhost docker-images]# docker ps 
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS              PORTS               NAMES
12c67284c170        docker.io/centos:centos7.4.1708   "/bin/bash"         38 minutes ago      Up 32 minutes                           serene_liskov
[root@localhost docker-images]# docker inspect -f {{.State.Pid}} 12c67284c170
7215
```

- 拿到该进程的`PID`之后，我们就可以使用`nsenter`命令访问该容器了

```
[root@localhost docker-images]# nsenter --target 7215 --mount --uts --ipc --net --pid
[root@12c67284c170 /]# ls
```

> 当然，如果每次输入那么多参数太麻烦的话，网上有许多做好的脚本，地址如下：[**http://yeasy.gitbooks.io/docker_practice/content/container/enter.html**](http://yeasy.gitbooks.io/docker_practice/content/container/enter.html)
>
> [**http://www.tuicool.com/articles/eYnUBrR**](http://www.tuicool.com/articles/eYnUBrR)

##### 使用`exec`命令进入Docker容器

- exec命令用于进入容器，这种方式相对更简单一些

```
[root@localhost docker-images]# docker ps 
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS              PORTS               NAMES
12c67284c170        docker.io/centos:centos7.4.1708   "/bin/bash"         46 minutes ago      Up 40 minutes                           serene_liskov
[root@localhost docker-images]# docker exec -it 12c67284c170 /bin/bash
[root@12c67284c170 /]#
```

##### 使用网上写好的.bashrc_docker

- 使用网上写好的`.bashrc_docker`,b并将内容放到`.bashrc`中

```
wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker; 
$ echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
```

- 这个文件中定义了很多方便使用的Docker命令，例如`docker-pid`可以获取某个容器的PID；而`docker-enter`可以进入容器或直接在容器内执行命令

```
$ echo $(docker-pid <container>)
$ docker-enter <container> ls
```

#### 导出和导入容器

##### 导出容器

- 如果想要导出本地某个容器，可以使用`docker export`命令，这样将导出的容器快照到本地文件。

```
[root@localhost docker-images]# docker ps 
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS              PORTS               NAMES
12c67284c170        docker.io/centos:centos7.4.1708   "/bin/bash"         53 minutes ago      Up 46 minutes                           serene_liskov
[root@localhost docker-images]# docker export 12c67284c170 > centos7.4.tar.gz
[root@localhost docker-images]# ls
centos7.4.tar.gz
```

##### 导入容器快照

- 可以使用`docker import`从容器快照文件中在导入镜像，例如：

```
[root@localhost docker-images]# cat centos7.4.tar.gz | docker import - test/centos:v7
sha256:765751a0205912a6186a42883baeabe339fc196989c3193bc8b1658339e0187a
[root@localhost docker-images]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test/centos         v7                  765751a02059        3 seconds ago       197 MB
```

- 也可以通过指定URL或者某个目录来导入，例如：

```
docker import http://example.com/exampleimage.tgz example/imagerepo
```

> **注意**：用户即可以使用`docker load`来导入镜像存储文件到本地镜像库，也可以使用`docker import`来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时快照状态）。而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

#### 删除容器

- 可以使用`docker rm`来删除一个处于终止状态的容器。例如：

```
[root@localhost docker-images]# docker ps -a
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS                       PORTS               NAMES
12c67284c170        docker.io/centos:centos7.4.1708   "/bin/bash"         About an hour ago   Exited (137) 5 seconds ago                       serene_liskov
[root@localhost docker-images]# docker rm 12c67284c170
12c67284c170
[root@localhost docker-images]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

- 如果要删除一个正在运行中的容器，可以添加`-f`参数，Docker会发送`SIGKILL`信号给容器。



## 3. Docker 仓库

- Docker 仓库用来保存镜像，是集中存放镜像文件的场所，可以理解为代码控制中的代码仓库。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分，实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)

- 仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub（https://hub.docker.com/），存放了数量庞大的镜像供用户下载。国内的公开仓库包括 Docker Pool等，可以提供大陆用户更稳定快速的访问。

- 当然，用户也可以在本地网络内创建一个私有仓库。当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像的时候，只需要从仓库上 pull 下来就可以了。

- 容器混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务上可以有多个仓库，而每个仓库下面有多个镜像，从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址：dockers.io/centos 来说，docker.io是注册服务器地址，centos是仓库名。大部分时候，并不需要严格区分这两者的概念

注：Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

### Docker Hub公共仓库

- 目前Docker官方维护了一个公共仓库Docker Hub(https://hub.docker.com/)，其中已经包括了超过15000个镜像，大部分需要，都可以通过Docker Hub中直接下载镜像来实现。

#### 登录

- 可以通过执行docker login 命令来输入用户名、密码和邮箱来完成注册和登录。注册成功后，本地用户目录的`.dockercfg`中将保存用户的认证信息

#### 基本操作

- 用户无需登录即可通过`docker search`命令来查找官方仓库中的镜像，并利用`docker pull`命令来将他们下载到本地。
- 例如以 centos 为关键词进行搜索：

```
[root@localhost ~]# docker search centos
INDEX       NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/centos                             The official build of CentOS.                   5902      [OK]       
docker.io   docker.io/ansible/centos7-ansible            Ansible on Centos7                              128                  [OK]
docker.io   docker.io/jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos ...   114                  [OK]
docker.io   docker.io/consol/centos-xfce-vnc             Centos container with "headless" VNC sessi...   111                  [OK]
docker.io   docker.io/centos/mysql-57-centos7            MySQL 5.7 SQL database server                   71                   
docker.io   docker.io/imagine10255/centos6-lnmp-php56    centos6-lnmp-php56                              58                   [OK]
........
```

- 可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、星级（表示该镜像的受欢迎程序）、是否官方创建、是否自动创建。官方的镜像说明是官方项目组创建和维护的。autometed资源允许用户验证镜像的来源和内容。

- 根据是否官方提供，可以将镜像资源分为两类。一种是类似centos这样的基础镜像，被称为`基础`或`根`镜像。这些基础镜像是有Docker公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。还有一种类型，比如`ansible/centos7-ansible`镜像，它是由Dokcer的用户创建并维护的。往往带有功能前缀或用户名前缀，可以通过前缀user_name来指定使用某个用户提供的镜像，或某个功能的镜像。
- 另外，在查找的时候通过 `-s N`参数可以指定仅显示评价为N星以上的镜像。
- 用户也可以在登录后通过docker push命令将镜像推送到Docker Hub

#### 自动创建

- 自动创建（Automated Builds）功能对于需要经常升级镜像内的程序来说，十分方便。有时候，用户创建了镜像，安装了某个软件，如果软件发布新版本则需要手动更新镜像。
- 而自动创建允许用户通过Docker Hub指定跟踪一个目录网站（支持GitHub或BitBucket等）上的项目，一旦项目发生新的提交，则自动执行创建。
- 要配置自动创建，包括如下的步骤：
  1. 创建并登录Docker Hub，以及目标网站；
  2. 在目标网站中连接账户到Docker Hub；
  3. 在Docker Hub中配置一个自动创建；
  4. 选取一个目标网站中的项目（需要含Dockerfile）和分支；
  5. 指定Dockerfile的位置，并提交创建；
  6. 之后可以在Docker Hub的自动创建页面中跟踪每次创建的状态；

#### 私有仓库

- 有使用使用Docker Hub这样的公共仓库不方便，用户可以创建一个本地仓库供私人使用。
- docker-registry是官方提供的功能，可以用户构建私有的镜像仓库。

##### 安装运行docker-registry

- 容器安装，在安装了Docker之后，可以通过获取官方registry镜像

```
[root@localhost ~]# docker pull registry
Using default tag: latest
Trying to pull repository docker.io/library/registry ... 
latest: Pulling from docker.io/library/registry
486039affc0a: Pull complete 
ba51a3b098e6: Pull complete 
8bb4c43d6c8e: Pull complete 
6f5f453e5f2d: Pull complete 
42bc10b72f42: Pull complete 
Digest: sha256:7d081088e4bfd632a88e3f3bcd9e007ef44a796fddfe3261407a3f9f04abe1e7
Status: Downloaded newer image for docker.io/registry:latest
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
docker.io/registry   latest              708bc6af7e5e        2 months ago        25.8 MB
```

- 创建容器registry

```
[root@localhost ~]# mkdir -p /www/docker/registry
[root@localhost ~]# docker run --name registry  \
>     --restart=always   \
>     -p 5000:5000     \
>     -v /www/docker/registry:/var/lib/registry   \
>     -d docker.io/registry
9f31b44b3ce26a5727f8b3999b7cd551654e6f8a2f511b7075720f1ba5ec4818
```

- 镜像仓库地址使用域名

```
[root@localhost ~]# echo "127.0.0.1  hub.test.com" >> /etc/hosts
```

- 修改tag

```
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos     centos7.4.1708      9f266d35e02c        12 months ago       197 MB
[root@localhost ~]# docker tag docker.io/centos:centos7.4.1708 hub.test.com:5000/centos:7.4.1708
[root@localhost ~]# docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos           centos7.4.1708      9f266d35e02c        12 months ago       197 MB
hub.test.com:5000/centos   7.4.1708            9f266d35e02c        12 months ago       197 MB
```

- 上传、删除、下载镜像

```
[root@localhost ~]# docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos           centos7.4.1708      9f266d35e02c        12 months ago       197 MB
hub.test.com:5000/centos   7.4.1708            9f266d35e02c        12 months ago       197 MB
[root@localhost ~]# docker push hub.test.com:5000/centos:7.4.1708    ##  上传镜像
The push refers to a repository [hub.test.com:5000/centos]
606d67d8e1b8: Pushed 
7.4.1708: digest: sha256:78b63c089c40a2ca0d85db9125fb3193c6ff74814bba5f894c803e5ae02a4035 size: 529
[root@localhost ~]# docker rmi hub.test.com:5000/centos:7.4.1708     ## 删除镜像
Untagged: hub.test.com:5000/centos:7.4.1708
Untagged: hub.test.com:5000/centos@sha256:78b63c089c40a2ca0d85db9125fb3193c6ff74814bba5f894c803e5ae02a4035
[root@localhost ~]# docker rmi docker.io/centos:centos7.4.1708
Untagged: docker.io/centos:centos7.4.1708
Untagged: docker.io/centos@sha256:8906d699cbd9406b07a105bedebc14a5945c200971b0a3a067aa245badc545b2
Deleted: sha256:9f266d35e02cc56fe11a70ecdbe918ea091d828736521c91dda4cc0c287856a9
Deleted: sha256:606d67d8e1b88099cf33bcbe675d0fb4f9035432f540a0d8be46017303c5cbe7
[root@localhost ~]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
[root@localhost ~]# docker pull hub.test.com:5000/centos:7.4.1708      ## 下载镜像
Trying to pull repository hub.test.com:5000/centos ... 
7.4.1708: Pulling from hub.test.com:5000/centos
840caab23da4: Pull complete 
Digest: sha256:78b63c089c40a2ca0d85db9125fb3193c6ff74814bba5f894c803e5ae02a4035
Status: Downloaded newer image for hub.test.com:5000/centos:7.4.1708
[root@localhost ~]# docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
hub.test.com:5000/centos   7.4.1708            9f266d35e02c        12 months ago       197 MB
```

- 查看仓库镜像

```
[root@localhost ~]# curl hub.test.com:5000/v2/_catalog
{"repositories":["centos"]}
```
