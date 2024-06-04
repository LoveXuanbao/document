# 一、Nexus手动安装

> 安装`Neuxs`的前置首要条件是存在jdk1.8以上的环境

## 1、java 安装

- 下载`jdk`

    - [jdk 下载地址](https://www.oracle.com/cn/java/technologies/javase-downloads.html)

- 安装`java`

    - 创建`java`目录

        ```sehll
        mkdir /usr/java
        ```

    - 解压下载好的文件到 `java`目录

        ```shell
        tar -zxvf server-jre-8u191-linux-x64.tar.gz -C /usr/java/
        ```

    - 设置环境变量

        ```shell
        vim /etc/profile
        
        export JAVA_HOME=/usr/java/jdk1.8.0_191
        export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
        export PATH=$PATH:$JAVA_HOME/bin
        ```


- 使 /etc/profile 配置文件生效

```shell
source /etc/profile
```

## 2、Nexus 安装

- 下载 `Neuxs`

    - [Nexus 官方网站](https://www.sonatype.com/download-oss-sonatype)

    - [Neuxs 最新版本地址](https://help.sonatype.com/repomanager3/download)

    - [Neuxs 以前版本地址](https://help.sonatype.com/repomanager3/download/download-archives---repository-manager-3)

        ```
        wget http://download.sonatype.com/nexus/3/nexus-3.22.1-02-unix.tar.gz
        ```

- 安装`Nexus`

    - 创建`Neuxs`安装目录

        ```shell
        mkdir -pv /data/nexus
        ```

    - 解压到安装目录

        ```shell
        tar -zxvf nexus-3.14.0-04-unix.tar.gz -C /data/nexus
        ```


        解压到安装目录之后，我们可以看到解压后通常有两个目录
    
    | 名称          | 作用                                                  |
    | ------------- | :---------------------------------------------------- |
    | nexus-x.x.x   | `Neuxs`运行所需要的文件，运行脚本，依赖jar包等        |
    | sonatype-work | 该目录包含`Nexus`生成的配置文件、日志文件、仓库文件等 |


​    

    - 创建 `Nexus`用户，或者使用`root`用户也可以
    
        ```shell
        useradd nexus
        ```
    
    - 设置用户环境变量
    
        ```shell
        vim /root/.bashrc
        
        NEXUS_HOME="/data/nexus/nexus-3.14.0-04"
        ```


​        

    - 设置`nexus`启动用户，不设置的话，默认使用root用户
    
        ```shell
        vi $NEXUS_HOME/bin/nexus.rc
        
        #run_as_user=""
        run_as_user="root"
        ```

- 把`nexus`启动命令加入到`system`中，`centos_6`使用下面[Nexus 优化配置](#Nexus 优化配置)中的方式

    ```shell
      vi /etc/systemd/system/nexus.service
      [Unit]
      Description=nexus service
      After=network.target
      
      [Service]
      Type=forking
      LimitNOFILE=65536
      ExecStart=/data/nexus/nexus-3.14.0-04/bin/nexus start
      ExecStop=/data/nexus/nexus-3.14.0-04/bin/nexus stop
      User=root
      Restart=on-abort
      
      [Install]
      WantedBy=multi-user.target  
    ```

      

    - 修改端口，根据默认修改

        ```shell
        cd /data/nexus/nexus-3.14.0-04/etc/
        vi nexus-default.properties
        ...
        application-port=8081
        application-host=0.0.0.0
        ...
        ```

    - 启动`nexus`并添加开机自启动，`centos_6`使用下面[Nexus 优化配置](#Nexus 优化配置)中的方式

        ```shell
        systemctl start nexus
        systemctl enable nexus
        ```

    - 访问`nexus`页面

        访问`web`页面，默认监听端口为`8081`，即访问http://localhosts:8081  并使用默认管理员账号`admin/admin123`登录

        ![image-20230825170358750](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170358750.png)

        出现上图页面，说明配置`Nexus`成功！点击右上角`"Log in"`，输入默认用户名`admin`，密码`admin123`进行登录。

# 二、Nexus优化配置

## 1、设置开机自启动

- `centos_6`设置开机自启动，`centos_7`就按照上方的配置即可

```shell
ln -s /data/nexus/nexus-3.14.0-04/bin/nexus /etc/init.d/nexus
chkconfig --add nexus
chkconfig nexus3 on
```

## 2、配置运行用户

- `Nexus`可以使用`root`运行，不过官方文档里边不建议使用`root`来运行，因此使用普通用户来运行
  
```shell
[root@nexus ~]$ useradd nexus
[root@nexus ~]$ cd /data/nexus/nexus-3.14.0-04/bin
[root@nexus bin]$ vim nexus.rc
run_as_uesr="nexus"
```

<font color=#FF0000>配置之后记得更改目录权限，否则下次启动会没有权限</font>

```shell
[root@nexus ~]$ chown -R nexus.nexus /data/nexus/nexus-3.14.0-04
[root@nexus ~]$ chown -R nexus.nexus /data/nexus/sonatype-work
```

## 3、配置`jdk`

- 如果这里不配置，一般会使用默认的`JAVA_HOME`的变量，如果系统中有多个，那么可以进行配置
  
```bash
[root@nexus ~]$ cd /data/nexus/nexus-3.14.0-04/bin
[root@nexus bin]$ vim nexus
修改第14行
INSTALL4J_JAVA_HOME_OVERRIDE=/usr/local/jdk1.8.0_144
```

## 4、修改端口

一般使用默认的，如果需要修改，则更改`/data/nexus/nexus-3.14.0-04/etc/nexus-default.properties`里边的配置

## 5、配置存储及日志位置

```shell
[root@nexus bin]$ vim /data/nexus-3.14.0-04/bin/nexus.vmoptions

一般都不做修改，使用默认即可，此处是为了了解这个知识点

-XX:LogFile=../sonatype-work/nexus3/log/jvm.log
-Dkaraf.data=../sonatype-work/nexus3
-Djava.io.tmpdir=../sonatype-work/nexus3/tmp
```

# 三、Nexus搭建本地yum源

## 一、创建Blob

![image-20230825170452058](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170452058.png)

填写`name`之后，path会自动在对应的目录创建blob目录

![image-20230825170531452](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170531452.png)

## 二、创建Repositories（仓库）

![image-20230825170614070](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170614070.png)

![image-20230825170645094](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170645094.png)

- `name`：仓库名称
- `Reoidate Depth`：目录层即，一般2层即可
- `Blob Store`：关联Blob设备

![image-20230825170720462](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170720462.png)

## 三、上传RPM包

```shell
[root@localhost package]# cat load-rpm.sh 
# 把当前目录下的rpm包通过curl载入nexus
set -euo pipefail

for rpm in `ls | grep ".*\.rpm$"`; do
        rpm_url="http://192.168.88.100:8081/repository/yum/centos/7/$rpm"
	curl --user 'admin:admin123' --upload-file $rpm $rpm_url
	echo $rpm
done
```

# 四、Nexus搭建阿里云yum代理

如果需要配置阿里云网络代理，需要`nexus`服务器能够访问外网，并且需要在网卡配置文件中添加如下配置文件

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33		该网卡根据自己访问外网的网卡来添加下列文件
***
***
DNS1=114.114.114.114
DNS2=8.8.8.8
systemctl restart network
```

> 阿里云`yum`源地址：http://mirrors.aliyun.com/centos
>
> 阿里云`epel`源地址：http://mirrors.aliyun.com/epel 

## 一、创建Blob

![image-20230825170753016](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170753016.png)

填写name之后path会自动创建blob目录

![image-20230825170827284](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170827284.png)

## 二、创建Repositories（仓库）

### 创建一个proxy类型的yum仓库

![image-20230825170905768](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170905768.png)

![image-20230825170941039](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170941039.png)

- `name`：代理仓库的名称
- `Remote Storage`：远程代理的地址，这里填写：http://mirrors.aliyun.com/centos
- `Blob store`：关联Blob
- 其他均为默认，可以创建多个proxy仓库，如163、搜狐等，根据个人需求创建

![image-20230825171015423](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171015423.png)

### 在创建一个proxy类型的yum仓库，代理epel源

创建方式同上，name和Remote Storage修改即可

### 创建一个group类型的yum仓库

- `name`：yum，自己定义
- `Storage`：选择yum专用的Blob存储
- `group`：将左边可选的仓库，添加到右边的members下

![image-20230825171124880](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171124880.png)

## 三、构建缓存

新建一台环境干净的主机，将yum 源指向到nexus私服中来

```shell
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# ls
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo
[root@localhost yum.repos.d]# mkdir bak
[root@localhost yum.repos.d]# mv CentOS-* bak
[root@localhost yum.repos.d]# ls
bak

添加如下配置
[root@localhost yum.repos.d]# vi nexus.repo
[nexus]
name=Nexus Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/os/$basearch/
enabled=1
gpgcheck=0

[updates]
name=updates Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/updates/$basearch/
enabled=1
gpgcheck=0

[extras]
name=extras Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/extras/$basearch/
enabled=1
gpgcheck=0

[centosplus]
name=centosplus Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/centosplus/$basearch/
enabled=1
gpgcheck=0

[epel]
name=epel Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/$basearch/
enabled=1
gpgcheck=0
```

清理缓存，并重新构建缓存

![image-20230825171204329](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171204329.png)

nexus页面查看group类型的yum仓库已经有内容了，然后就可以愉快的去安装包了

![image-20230825171241199](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171241199.png)



# 五、Nexus代理docker的yum源

## 1、创建代理的yum仓库，并填写代理地址：

```
https://mirrors.aliyun.com/docker-ce/linux/centos/
```

![image-20230825171319458](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171319458.png)

## 2、在 yum_group 组中，把新创建的代理仓库添加进去，从左边移动到右边

![image-20230825171351100](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171351100.png)

## 3、然后在服务器的/etc/yum.repos.d/xxx.repo文件中添加以下内容：

```
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum//$releasever/$basearch/stable
enabled=1
gpgcheck=0
```

## 4、然后清空缓存，重新创建缓存

```
yum clean all
yum makecache fast
```

# 六、Neuxs 配置docker 仓库

## 1、创建 Blob 块

> 为了仓库数据得独立性和安全性，我们给docker创建一个独立得Blob块存储

- **Name：**填写 Blob 块存储得名称
- **Path：**会自动生成并补全，默认在Nexus安装目录下面得 sonatype-work/nexus3/blobs/ 目录下，也可以修改到其他目录或磁盘，甚至可以是 NFS 或者 cephfs 得目录。

![image-20230825171424634](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171424634.png)

## 2、创建一个 hosted 类型的 docker 仓库

- Hosted 类型得仓库用作我们得私有仓库，点击 Repository 下面得 Repositories --> Create repository --> docker(hosted)
  - **Name：**输入一个简单直观得名字
  - **Online：**勾选。这个开关可以设置这个Docker Repo是在线还是离线
  - **Repository Connectors：**下面包含 HTTP 和 HTTPS 两种类型得 port。连接器允许 docker 客户端直接连接到 docker 仓库，并实现一些请求操作。如 docker pull、docker push、API 查询等。因此我们勾选 http 并填写端口 8445，Nexus就会启动一个监听到 8445 端口得连接器。
  - **Fore basic authentication：**勾选，这样得话就不允许匿名访问了，执行 docker pull 或 docker push 之前，都要先执行 docker login xxxx 来登录仓库
  - **Docker Registry API Support：**Docker registry 默认使用得是 API v2，但是为了兼容性，我们可以勾选启用 API v1
  - **Storage：**Blob store，我们下拉选择前面创建好的 Blob
  - **Hosted：**开发环境，我们运行重复发布，因此 Deployment policy 我们选择 Allow redeploy

![image-20230825171509016](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171509016.png)

## 3、创建一个 proxy 类型的 docker 仓库

- proxy 类型的仓库，可以帮助我们访问不能直接到达的网络，如另一个私有仓库或者外网的公共仓库，如 Google Cloud Registry
- 对于代理 Docker Hub，官方有简要的文档可以参考**https://help.sonatype.com/repomanager3/formats/docker-registry/proxy-repository-for-docker**

- 创建方法：与创建 hosted 类型的方式类似，点击 Repository 下面的 Repositories --> Create repository -- docker(proxy)
  - **Name：**输入一个简单直观的名字
  - **Online：**勾选，设置 Docker repo 是在线还是离线
  - **Repository Connectors：**不设置
  - **Proxy：**Remote Storage 当配置的 docker hub 的 proxy 时，这里填写【https://registry-1.docker.io】；
  - **Docker Index：**上一步 proxy 配置的 docker hub，点选 Use Docker Hub，或者选其他的填写 https://index.docker.io
  - **Storage：**Blob store，选择我们前面创建好的 blob

![image-20230825171553895](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825171553895.png)

![image-20230825175126457](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175126457.png)

## 4、创建一个 group 类型的 docker 仓库

- group 类型的 docker 仓库，是一个聚合类型的仓库。它可以将前面我们创建的仓库聚合成一个 URL 对外提供服务，可以屏蔽后端的差异性，实现类似透明代理的功能。

- 创建方法：点击 Repository 下面的 Repositories --> Create repository -- docker(group):
  - **Name：**输入一个简单直观的名字
  - **Online：**勾选，设置 Docker repo 是在线还是离线
  - **Repository Connectors：**勾选 http 并填写端口 8444（自定义），Nexus就会启动一个监听 8444 端口的连接器。
  - **Storage：**选择专用的 blob 存储 docker-hub-hosted
  - **group：**将左边需要添加的仓库，添加到右边的 members 下

![image-20230825175206000](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175206000.png)

![image-20230825175242159](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175242159.png)

## 5、配置客户端和项目以使用Nexus存储库

- 首先，先在/etc/hosts 添加主机映射，这样在本机拉取的时候，可以使用假域名的方式拉取，其他 docker 想要拉取，可以使用ip:port方式，也可以配置 /etc/hosts，使用假域名的方式
- 配置docker；修改 /etc/docker/daemon.json 并重启 docker
- 如果配置 http，那么 daemon.json 需要配置 刚才我们创建的时候指定的端口

```
[root@localhost ~]# vim /etc/hosts
...
192.168.1.1  repository.test.com
[root@localhost ~]# vim /etc/docker/daemon.json
{
    "insecure-registries": [
        "repository.test.com:8444",
        "repository.test.com:8445"
    ]
}
```

- 如果不使用 SSL 证书和域名，那么可能会遇到：**failed with status：401 Unauthorized**，解决办法就是在Neuxs点击设置 --> Security --> Realms --> Docker Bearer Token Realm

![image-20230825175319100](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175319100.png)

## 6、docker 登录仓库

- 在服务器上登录仓库，这里需要登录两次，以 8444 和 8445 为例

```
docker login repository.test.com:8444   # 输入nexus的用户名和密码
docker login repository.test.com:8445   # 输入nexus的用户名和密码
```

## 7、拉取Nginx镜像测试

```
[root@localhost docker]# docker pull repository.test.com:8444/nginx
Using default tag: latest
latest: Pulling from nginx
a076a628af6f: Pull complete 
0732ab25fa22: Pull complete 
d7f36f6fe38f: Pull complete 
f72584a26f32: Pull complete 
7125e4df9063: Pull complete 
Digest: sha256:0b159cd1ee1203dad901967ac55eee18c24da84ba3be384690304be93538bea8
Status: Downloaded newer image for repository.test.com:8444/nginx:latest
repository.test.com:8444/nginx:latest
[root@localhost docker]# docker images
REPOSITORY                                                  TAG           IMAGE ID       CREATED         SIZE
repository.test.com:8444/nginx                              latest        f6d0b4767a6c   2 weeks ago     133MB
```



# 七、Nexus上传文件的方法

## 1、批量上传RPM包

```shell
[root@localhost package]# cat load-rpm.sh 
# 把当前目录下的rpm包通过curl载入nexus
set -euo pipefail

for rpm in `ls | grep ".*\.rpm$"`; do
        rpm_url="http://192.168.88.100:8081/repository/yum/centos/7/$rpm"
 curl --user 'admin:admin123' --upload-file $rpm $rpm_url
 echo $rpm
done
```

## 2、上传单个文件

```shell
curl -v --user 'admin:admin123' --upload-file packagename http://192.168.88.100:8081/repository/[路径]/packagename
```

## 3、上传整个目录

```shell
find . -type f -exec curl -v --user 'admin:admin123' --upload-file {} http://192.168.88.100:8081/repository/yum/{} \;
```

# 八、Nexus清理释放磁盘空间

## 应用背景

自建的`maven`私服（或者叫私仓）`nexus`在使用过程中，因很多服务不断迭代更新上传文件至`nexus`中，底层存放在一个叫`Blob Stores`的存储中，最近发现该存储已增大至好几百G，有必要清理一下，释放磁盘空间。

## 测试环境

| 操作系统   | 应用            |
| ---------- | --------------- |
| Centos 7.4 | nexus-3.14.0-04 |

## 操作步骤

1. 在`nexus`界面清理对应的旧版本或者想要清理的应用包

   如图示：

   ![image-20230825175351614](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175351614.png)

   注意：在删除多个目标后，会发现实际物理磁盘并没有释放出来，是因为在后台只是被标记为`deletion`，就好比你用`delete`语句删除`mysql`中的条目时，磁盘空间不会释放出来一样，因此，还需要第二步操作。

2. 创建定时任务

   这里会创建一个定时任务，任务类型为Compact Blobstore，然后填写定时任务详情，如下：

   ![image-20230825175428886](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175428886.png)

   创建完成，页面跳转至如下页面：

   ![image-20230825175503881](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175503881.png)

   因为我们创建的是手动执行的，所以点击进去，run是执行任务，delete task是删除该任务

   ![image-20230825175537834](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825175537834.png)