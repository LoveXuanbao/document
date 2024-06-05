## gitlab创建用户新建项目提交代码

- gitlab安装完成后首次在浏览器打开web页面，会出现root初始密码得界面，密码设置8位数，生产建议使用复杂度高得强密码

- 登录成功后，点击右上角得**New project --> Create blank project**填写项目名称，项目由三个权限
  - **Private**： 私有项目，只有自己或者项目组内得人才能访问
  - **Internal**：所有登录得用户都能访问
  - **Public**：公开得，任何人都能访问

![image-20230825163446418](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163446418.png)

- 新建成功后，把项目地址复制出来

![image-20230825163547531](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163547531.png)



## 安装git

**git**不仅仅是个版本控制系统，也是个内容管理系统（CMS），工作管理系统等。

### git和svn得区别

- git是分布式得，svn不是，这是git和其他非分布式得版本控制系统，例如svn，cvs等最核心的区别；
- git把内容按元数据方式存储，而svn是按文件，所有的资源控制系统都是把文件的元信息隐藏在一个类似**.svc、.vcs**等文件夹里；
- git分支和svn的分支不同，分支在svn中就是版本库中的另外一个目录；
- git没有一个全局的版本号，而svn有，目录为止这是跟svn项目git缺少的最大的一个特征；
- git的内容完整性要优于svn，git的内容存储使用的是SHA-1哈希算法，这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。



### yum 安装 git

```bahs
yum install -y git
```



### 查看git版本号

```bash
[root@localhost ~]# git --version
git version 1.8.3.1
```



### 添加git全局配置

```bash
[root@localhost ~]# git config --global user.name "xxx"
[root@localhost ~]# git config --global user.email "xxxx@xx.com"
[root@localhost ~]# git config --list
user.name=xxx
user.email=xxxx@xx.com
```



### 创建一个新仓库

```bash
[root@localhost ~]# git clone ssh://git@192.168.88.101:10022/root/test.git
正克隆到 'test'...
The authenticity of host '[192.168.88.101]:10022 ([192.168.88.101]:10022)' can't be established.
ECDSA key fingerprint is SHA256:QtJc4X/0Z2k1InxGLWeEWsyIXQYeaqZui+QF1Te8yKo.
ECDSA key fingerprint is MD5:ca:1f:55:aa:78:fb:3a:03:b9:ed:7b:65:7c:6b:ba:52.
Are you sure you want to continue connecting (yes/no)? yes   ## 第一次拉取需要确认
Warning: Permanently added '[192.168.88.101]:10022' (ECDSA) to the list of known hosts.
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
接收对象中: 100% (3/3), done.
[root@localhost ~]#
```



### 编写一个新文件

```bash
[root@localhost ~]# cd test/    ## 进入目录
[root@localhost test]# ll    ## 查看项目中的文件
总用量 4
-rw-r--r-- 1 root root 8 2月   5 18:03 README.md
[root@localhost test]# touch test.txt   ## 创建一个新文件
[root@localhost test]# ll
总用量 4
-rw-r--r-- 1 root root 8 2月   5 18:03 README.md
-rw-r--r-- 1 root root 0 2月   5 18:04 test.txt
[root@localhost test]#
```



### 把工作区中被修改过的文件添加到暂存区

```bash
[root@localhost test]# git add .
```



### 把暂存区的内容提交到版本库里

```bash
[root@localhost test]# git commit -m "添加 README.md 文件"
[master（根提交） 4e75624] 添加 README.md 文件
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```
