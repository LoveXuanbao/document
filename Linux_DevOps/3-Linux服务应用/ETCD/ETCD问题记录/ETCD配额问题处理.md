## ETCD配置问题处理与验证

- 报错信息：

  > 容器创建的时候，报错：Error etcdserver mvcc database space exceeded

  ![image-20230825162857241](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162857241.png)

- 问题原因：

  k8s创建pod的时候，会向etcd中写数据，etcd官方默认的空间配额是2G，最大支持8G，以下是官方下载包内的说明，

  ```
  The default storage size limit is 2GB, configurable with `--quota-backend-bytes` flag. 8GB is a suggested maximum size for normal environments and etcd warns at startup if the configured value exceeds it.
  ```

  所有的运维管理都在操作ETCD的存储空间；存储空间的配额用于控制ETCD数据空间的大小，如果ETCD节点磁盘空间不足了，配额回触发告警，然后ETCD系统将进入操作受限的维护模式。

### 解决方案

- 先上传etcdctl命令包到etcd服务器上，并给予可执行权限
- ![image-20230825162941262](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825162941262.png)

- 第一种方式压缩

```
#使用API3
$ export ETCDCTL_API=3 
# 获取当前版本
$ rev=$(etcdctl --endpoints=http://127.0.0.1:2379 endpoint status --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9].*')
# 压缩掉所有旧版本
etcdctl --endpoints=http://127.0.0.1:2379 compact $rev
# 整理多余的空间
etcdctl --endpoints=http://127.0.0.1:2379 defrag
# 取消告警信息
etcdctl --endpoints=http://127.0.0.1:2379 alarm disarm
```

- 第二种方式压缩

```
# 查看节点状态
ETCDCTL_API=3 ./etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
endpoint status --write-out="table"
## --write-out="table"是输出的格式，可以是json，可以是table，默认是simple，这个参数可以不加

# 获取旧版本号
ETCDCTL_API=3 ./etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
endpoint status --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9]*'

# 压缩旧版本
ETCDCTL_API=3 etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
compact 113664336 
## 113664336 为上一步获取到的旧版本号

# 清理碎片
ETCDCTL_API=3 ./etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
defrag

# 查看告警
ETCDCTL_API=3 ./etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
alarm list

# 取消告警
ETCDCTL_API=3 ./etcdctl \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
--endpoints=https://127.0.0.1:2379 \
alarm disarm


```



- 修改启动文件

```
...
--auto-compaction-retention=1       ## 表示每隔一小时自动压缩一次
--quota-backend-bytes=8388608000    ## 磁盘空间调整为8G，官方建议最大8G
```

然后重启etcd即可