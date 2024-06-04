Rclone是一个开源的命令行程序，用于管理云存储上的文件

## Rclone官网

`https://rclone.org/`

## Rclone文档

`https://rclone.org/docs/`



使用Rclone实现minio数据的迁移

- 使用场景：网络通畅，不同服务器之间迁移、云存储系统迁移、支持分布式存储
- 特性：使用需要安装rclone程序；安全，边界，可维护性高



## 安装

```bash
# 安装EPEL源
yum -y install epel-release

# 安装基本组件和依赖
yum -y install wget unzip screen fuse fuse-devel

# 下载Rclone
wget https://downloads.rclone.org/rclone-current-linux-amd64.zip

# 解决安装包
unzip rclone-current-linux-amd64.zip

# 给rclone文件添加可执行权限
cd ./rclone-*
chmod 755 rclone

# 移动可执行文件到/usr/bin目录下
cp rclone /usr/bin/
```



## 配置

```bash
mkdir -p /root/.config/rclone/
vim /root/.config/rclone/rclone.conf
[minio]
type = s3
provider = Minio
evn_auth = false
access_key_id = minioadmin
secret_access_key = s2Cpi9NvZc3Vuvw6
endpoint = http://11.53.101.136:9000
```



## 备份

```bash
rclone copy minio: /data/sdb1/minio_bak/ --checksum
```



## 其他节点导入

- 同样需要先安装rclone工具

```bash
mkdir -p /root/.config/rclone/
vim /root/.config/rclone/rclone.conf
[minio]
type = s3
provider = Minio
evn_auth = false
access_key_id = minioadmin     # 新minio的用户名
secret_access_key = s2Cpi9NvZc3Vuvw6    # 新minio的密码
endpoint = http://11.53.101.136:9000   # 新minio的地址
```