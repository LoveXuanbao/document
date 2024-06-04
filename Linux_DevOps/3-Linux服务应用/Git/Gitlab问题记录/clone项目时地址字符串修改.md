内网服务器搭建gitlab环境，搭建成功后，在gitlab新建了一个项目，然后clone项目时的地址如下：

```bash
git@0096ce63c43f:root/jump.git
或者这样
http://0096ce63c43f/root/jump.git
```



### 处理办法

1. 如果挂载的配置文件在宿主机上，启动命令如下

```bash
docker run -d --name gitlab \
    --restart always \
    -p 8880:80 \
    -p 443:443 \
    -p 10022:22 \
    -v /data/gitlab/config:/etc/gitlab \
    -v /data/gitlab/data:/var/opt/gitlab \
    -v /data/gitlab/logs:/var/log/gitlab \
    gitlab/gitlab-ce:latest
```

2. 那么可以直接查找文件

```bash
find /data/gitlab/ -name gitlab.rb
```

3. 然后修改文件

```bash
vim /data/gitlab/config/gitlab.rb
```

4. 在文件中新增活修改如下配置。值为**http://gitlab所在服务器的IP**，需要注意的时，不加端口！

```bash
external_url 'http://192.168.88.101'
```

![image-20230825163049991](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825163049991.png)