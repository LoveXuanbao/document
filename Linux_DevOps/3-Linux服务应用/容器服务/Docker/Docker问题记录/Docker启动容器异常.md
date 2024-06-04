## docker 启动容器异常
### 问题描述
- docker服务异常停止，重启docker后，容器启动失败

### 错误信息
- Error response from daemon: OCI runtime create failed: container with id exists: xxx unknown

![image-20230825145450840](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145450840.png)

### 错误原因
- docker启动的时候，会在运行目录（/var/run/docker/runtime-runc/moby）（不同环境，可能目录不一样，可以通过`find / -name '容器ID'` 查找）下生成以docker-ID，因为docker异常停止，改容器文件并没有删除，所以启动的时候，会报错该容器已存在

### 解决办法
```
find / name "报错的容器ID"
cd /var/run/docker/runtime-runc/moby
rm -rf "刚才查找到报错的容器ID"
docker start "有问题的容器"    ## 重新启动容器
```