# pip国内加速

## 一、创建pip配置文件目录

```bash
mkdir ~/.pip
```

## 二、编译pip配置文件

- 配置阿里云pip源

```bash
vim ~/.pip/pip.conf
[global]
timeout=120
trusted-host=mirrors.aliyun.com 
index-url=http://mirrors.aliyun.com/pypi/simple/ 
```

> 参数说明：
>
> timeout： 超时时间，避免因阻塞导致下载失败；
>
> trusted-host：添加指定源为可信任主机；
>
> index-url：指定源的URL地址



# 国内常用加速源

```bash
清华大学：https://pypi.tuna.tsinghua.edu.cn/simple 

阿里云：http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学：http://pypi.hustunique.com/

山东理工大学：http://pypi.sdutlinux.org/

豆瓣：http://pypi.douban.com/simple/

```

