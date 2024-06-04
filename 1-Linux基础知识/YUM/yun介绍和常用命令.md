# yum基础介绍

## 一、Yum介绍

yum: 全程为 Yellow dog Updater，Modified，是一个在 Fedora 和 RedHat 以及 CentOS 中的 Shell 前端软件包管理器，基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖管理，并且一次安装所有依赖的软件包，无需繁琐地一次次下载、安装。yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简介而又好记。

> `软件包来源`:

 可供 yum 下载的软件包包括 Fedora 本身的软件包以及源自 rpmfusion 和 rpm. 的 Fedora Extras，全部是由 Linux 社区维护的，并且基本是由自由软件，所有软件包都有一个独立的 PGP 签名，主要是为了系统安全。

> `yum工作原理`:

执行 yum 命令时，会首先从 /etc/yum.repo.d目录下的众多 repo 文件中取得软件仓库的地址并下载"元数据"，"元数据"包含注册于该软件仓库内的所有软件包的包名及其所需的依赖环境等信息，yum 得到这些信息后会和本地环境做对比，进而列出确认需要安装哪些包，并在用户确认后开始安装。"元数据"由位于 yum 源服务器相关路径的 repodata 目录下的 repomd.xml 做索引

> `yum工作流程`: 

  服务端: 在服务器上存放了所有的 rpm 软件包，然后以相关的功能分析每个 rpm 文件的依赖关系，将这些数据记录成文件存放在服务器的某个特定目录内。

  客户端: 如果需要安装某个软件shi时，先下载服务器上面记录的依赖性关系文件(可通过本地、http或FTP方式)，通过对服务器端下载的记录数据进行分析，然后取得所有相关的软件，一次全部下载进行安装

## 二、Yum配置文件的基本路径

```
/etc/yum.repos.d/CentOS-Base.repo -->这个是库的基本路径
/etc/yum.conf -->这个是yum的基本配置
/etc/yum.pluginconf.d/ -->这个是yum插件的路径
如果想要安装插件可以用命令yum list yum-plugin*来搜索
会返回很多插件如yum-plugin-fastestmirror.noarch，就可以yum安装插件了
```

## 三、Yum的使用

```
yum的命令形式一般如：yum [-options] [command] [package...]
其中[options]是可选的，[command]为所要进行的操作，[package...]是操作的对象
常用选项和参数详见yum选项和参数
```

## 四、Yum配置文件说明

yum的一切信息都存储在一个叫yum.repo.d目录下的配置文件中，通常位于/etc/yum.repo.d/目录下。在这个目录下有很多文件，都是.repo结尾的，repo文件是yum源(也就是软件仓库)的配置文件，通常一个repo文件定义了一个或者多个软件仓库的细节内容，例如我们从哪里下载需要安装或者升级的软件包，repo文件中的设置内容将被yum读取和应用

>  vim /etc/yum.conf

![image-20230825144650059](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144650059.png)

```
cachedir=/var/cache/yum/ ##yum下载的RPM包的缓存目录，yum在此存储下载的rpm包和数据库
keepcache=0 ##缓存是否保存，1表示安装后保留软件包，0表示安装后删除软件包。
debuglevel=2 ##调试级别(0-10)，默认为2(具体调试级别的应用，百度、Google)
logfile=/var/log/yum.log ##存放系统更新软件的日志的目录。用户可以到该指定的文件中查询自己在过去的日子里都做了哪些更新
exactarch=1 ##有两个选项1和0，代表是否只升级和你安装软件包CPU体系一致的包，如果设为1，则如你安装了一个i386的rpm包，则yum不会用i686的包来升级。
obsoletes=1 ##这是一个update的参数，简单的说就是相当于upgrade，允许更新陈旧的RPM包，具体请google、百度。
gpgcheck=1 ##是否检查gpg(Gun Private Guard)，一种秘钥方式签名。
plugins=1 ##是否允许使用插件，默认是0不允许，但是我们一般会用yum-fastestmirror这个插件
installonly_limit=5 ##允许保留多少个内核包
distroverpkg=Centos-release ##指定一个软件包，yum会根据这个包判断你的发型版本，默认是RedHat-release，也可以使安装的任何针对自己发行版的rpm包。
```

> `vim /etc/yum.repo.d/XXX.repo`

![image-20230825144737935](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144737935.png)

```
[extras] --->这个表示的是名称，yum的ID，必须唯一，本地有多个yum源的时候，这里必须是唯一的。
name=CentOS-$releasever - Extras --->具体的yum源的名字，其实相当于对它的描述信息，$releasever，你可以使用这个变量参考红帽企业Linux发行版，也就是说表示当前发行版的大版本号。
mirrorlist --->是镜像服务器的地址列表，里边有很多的服务器地址。这里有一个变量$arch,cpu体系，还有一个变量：$basearch，cpu的基本体系组。
baserul --->是镜像服务器地址，只能写具体的确定地址；baseurl和mirrorlist都是指向yum源的地址，不同点是包含地址的多少，你若自己写的话，我们一般只写一个地址，直接baseurl就行
file:// -->指定本地目录，本地源
http:// -->指定网络上的位置，本地局域网源|网络源
ftp:// -->同上
gpgcheck=0 -->要不要验证，0不验证，1使用公钥检验rpm的正确性。
gpgcheck若是1，将对下载的rpm进行gpg的校验，校验秘钥就是gpgkey，一般自己的yum源是不需要检测的。
gpgcheck=0，那么gpgkey就可以不填写
gpgkey= -->如果上述检查签名，此处必须写(指定秘钥位置)，如果不检查，可以不写
enabled=1 -->定义此yum源是否启用，0不启用，1启用，默认为1
```

## 五、`.repo`文件中 `mirrorlist`和`baserul`的区别

mirrorlist: 就是一个表

mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os

其中 $releaserver 版本号，如果是centos-6，就 $releaserver 换成6代替，而 $basearch 是体系结构，就是i386 或这 X86_64

那么就可以把这个链接

http://mirror.centos.org/?relase=6&arch=x86_64&repo=os

输入浏览器得到

![image-20230825144824186](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144824186.png)

# yum常用命令

## 一、列举包文件

```shell
yum list ##列出资源库中所有可以安装或更新的rpm包；
yum list updates ##列出资源库中所有可以更新的rpm包；
yum list installed ##列出已经安装的所有rpm包；
yum list extras ##列出已经安装的但是不包含在官方资源库中的rpm包，例如安装了epel源的rpm包会列出来；
```

## 二、列举资源信息

```shell
yum info ##列出资源库中所有可以安装或更新的rpm包的信息；
yum info perl ##列出perl包信息；
yum info perl* ##列出perl开头的所有包的信息；
yum info updates ##列出资源库中所有可以更新的rpm包的信息；
yum info installed ##列出已经安装的所有的rpm包的信息
yum info extras ##列出已经安装的但是不包含在资源库中的rpm包的信息；
```

## 三、搜索

```shell
yum search packagename ##搜索匹配特定字符的rpm包，在包名称、包描述符等中搜索；
yum provides packagename ##反查包含特定文件名的rpm包，查询命令用yum provides */ifconfig，查询文件无需*/也可用yum wharprovides
```

## 四、管理包

> 1、安装rpm包，可以用通配符`*`来匹配

```shell
yum install packagename ##安装packagename包；
yum remove packagename ##会删除packagename包，以及相关依赖的包；
```

> 2、软件组管理

```shell
yum groupinstall "Chinese Support" ##安装指定的组；
yum groupupdate "Chinese Support" ##安装了的组成员软件包更新；
yum grouplist ##安装了的组和可以安装的组一览显示；
yum groupremove "Chinese Support" ##删除指定的组；
yum groupinfo "Chinese Support" ##指定组包含的软件包显示；
```

## 五、更新

```shell
yum check-update ##检查可更新的rpm包；
yum update ##更新所有的rpm包；
yum update kernel kernel-source ##更新指定的rpm包，如更新kernel和kernel-source；
yum upgrade ##大规模的版本升级，与yum update不同的是，连旧的淘汰的包也升级；
```