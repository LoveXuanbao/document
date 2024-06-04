## 搭建本地yum源

1、环境，准备一台Linux服务器

![image-20230825144031503](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144031503.png)

2、上传iso镜像包到该服务器的某个目录下，并创建挂载点目录

![image-20230825144100084](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144100084.png)

3、在/etc/fstab添加以下内容

`/media/CentOS-7-x86_64-DVD-1708.iso /media/cdrom iso9660 loop 0 0`

![image-20230825144239668](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144239668.png)

4、在执行命令`mount -a`使`/etc/fstab`配置文件生生效，然后`df -Th`查看挂载信息

![image-20230825144330261](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144330261.png)

5、进入yum源配置文件存放目录 `cd /etc/yum.repo.d/`创建个目录把系统自带的reoi文件备份起来，## 如果服务器可以连接外网，这些repo文件是可以直接使用的。

![image-20230825144410976](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144410976.png)

6、创建自己的yum源文件vim aa.repo 文件名可以自己随意起，但是后缀必须以.repo结尾，添加如下内容，

> 注意：baseurl后面的路径，写的是镜像挂载路径

![image-20230825144502961](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144502961.png)

7、先清除一遍缓存，在创建缓存，看到如下元数据缓存已建立，就代表成功，可以使用yum装rpm包了

![image-20230825144547925](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144547925.png)