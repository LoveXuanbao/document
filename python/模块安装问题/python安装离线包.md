# 在外网环境先下载好

- 为了不被现有环境冲突，使用虚拟环境打包

# 安装虚拟环境

```bash
pip install virtualenv
```

## 创建虚拟环境存放目录

```bash
cd /data
cirtualenv test
```

## 激活环境

```bash
cd test
source ./bin/activate
(test) [root@ecs-82801 venv]#     # 发现命令提示符最前边有（虚拟环境名称），即代表进入到了虚拟环境
```

## 使用：

- 进入环境后，所有的操作和正常使用python，pip是一样的

## 退出环境

```bash
(venv) [root@ecs-82801 venv]# deactivate 
[root@ecs-82801 venv]#
```

## 删除虚拟环境

```bash
# 先切换到虚拟环境目录以外，不要在虚拟环境目录下
[root@ecs-82801 venv]# cd ..
[root@ecs-82801 data]# ll
total 12
drwxr-xr-x 6 root          root 4096 May 10 16:43 venv
[root@ecs-82801 data]# pwd
/data

# 执行下面命令切换到虚拟换外的目录然后直接删除虚拟环境目录即可
[root@ecs-82801 data]# rm -rf venv/
```



# 在虚拟环境中安装我们要离线的包

- `注意：以下操作都在虚拟环境中执行`

## 一、创建一个packages目录，用来存放我们下载的包

```bash
(venv) [root@ecs-82801 venv]# mkdir packages
(venv) [root@ecs-82801 venv]# ll
total 20
drwxr-xr-x 3 root root 4096 May 10 16:18 bin
drwxr-xr-x 3 root root 4096 May 10 16:16 lib
drwxr-xr-x 3 root root 4096 May 10 16:16 lib64
drwxr-xr-x 2 root root 4096 May 10 16:27 packages
-rw-r--r-- 1 root root  206 May 10 16:16 pyvenv.cfg
```

## 二、安装我们需要的包

```bash
pip3 install django==1.8.6
pip3 install django-cors-headers==2.4.0
pip3 install jieba==0.38
pip3 install gensim==3.6.0
pip3 install uwsgi==2.0.17.1
pip3 install pandas==0.23.1
pip3 install py2neo==4.2.0
```

## 三、查看安装包

```bash
(venv) [root@ecs-82801 venv]# pip3 freeze
certifi==2023.5.7
cffi==1.15.1
click==8.0.4
colorama==0.4.5
cryptography==40.0.2
Django==1.8.6
django-cors-headers==2.4.0
gensim==3.6.0
idna==3.4
importlib-metadata==4.8.3
ipaddress==1.0.23
jieba==0.38
neobolt==1.7.17
neotime==1.7.4
numpy==1.19.5
pandas==0.23.1
prompt-toolkit==2.0.10
py2neo==4.2.0
pycparser==2.21
Pygments==2.3.1
pyOpenSSL==23.1.1
python-dateutil==2.8.2
pytz==2023.3
scipy==1.5.4
semantic-version==2.10.0
setuptools-rust==1.1.2
six==1.16.0
smart-open==6.3.0
typing_extensions==4.1.1
urllib3==1.24.3
uWSGI==2.0.17.1
wcwidth==0.2.6
zipp==3.6.0
```

## 四、生成requirements.txt文件

```bash
pip3 freeze > requirements.txt

(venv) [root@ecs-82801 venv]# ll
total 24
drwxr-xr-x 3 root root 4096 May 10 16:18 bin
drwxr-xr-x 3 root root 4096 May 10 16:16 lib
drwxr-xr-x 3 root root 4096 May 10 16:16 lib64
drwxr-xr-x 2 root root 4096 May 10 16:35 packages
-rw-r--r-- 1 root root  206 May 10 16:16 pyvenv.cfg
-rw-r--r-- 1 root root  560 May 10 16:31 requirements.txt   # 会在当前目录下生成该文件
```

## 五、将所有的包下载到执行的目录

```bash
pip3 wheel -w ./packages -r requirements.txt
pip3 download -d ./packages -r requirements.txt
```

> 上面两条命令说明：
>
> - wheel分析requirements文件，将所有的包及其依赖包下载为wheel格式，通过 -w 选项导入到当前目录下的packages目录中；wheel方式下载会将下载的包放入wheel缓存中，但是缺点是不可以下载源码包；
> - download分析requirements文件，将所有包进行下载，通过 -d 选项导入到当前目录下的packages目录中；download命令会查看wheel缓存，然后再去pypi下载，但download命令下载的包不会进入wheel缓存，download可以下载源码包
> - download适合补充wheel不可下载的包，两者搭配使用，才能将requirements.txt文件中的所有包完成的下载

`注意：如果只使用download方法下载，可能会安装失败报错`

## 六、打包下载好的文件放到离线服务器上

```bash
# 打包，在打包的时候，记得把requirements.txt 文件也打包进去
(venv) [root@ecs-82801 venv]# tar -zcvf packages.tar.gz packages/ requirements.txt 
```

# 离线服务器安装

- 在离线服务器上解压缩刚才上传的包

```bash
[root@gs-server-13896 python]# tar -zxf packages.tar.gz 
[root@gs-server-13896 python]# ll
总用量 94832
drwxr-xr-x 2 root root     4096 5月  10 16:46 packages
-rw-r--r-- 1 root root 97097920 5月  10 16:47 packages.tar.gz
-rw-r--r-- 1 root root      560 5月  10 15:52 requirements.txt



pip3 install --no-index --find-links=./ -r requirements.txt
```

- 在离线服务器上安装

```bash
pip3 install --no-index --find-links=./packages -r requirements.txt
```

> 命令说明：
>
> --no-index：忽略包索引，必须搭配 --find-links使用
>
> --find-links：用于指定包的下载连接，可以跟一个URL或者一个本地目录，一般常用于本地离线安装