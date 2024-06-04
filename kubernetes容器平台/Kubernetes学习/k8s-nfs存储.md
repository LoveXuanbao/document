## KUBERNETES 存储 NFS 部署
--------------------
##### 1、部署和配置NFS服务器
- 查看是否安装nfs: yum list installed | grep nfs
- 创建共享目录: mkdir -p /var/nfs/share && chmod 777 /var/nfs/share
- 修改NFS配置

```
# vim /etc/exports

 /var/nfs/share *(rw,insecure,sync,no_subtree_check,no_root_squash)
```
##### 2、启动NFS相关服务
```
# systemctl enable rpcbind
# systemctl start rpcbind
# systemctl enable nfs-server
# systemctl start nfs-server
# exportfs

# cd /var/nfs/share
# touch nfs-test #创建测试文件
```

##### 3、在kubernetes node 安装 nfs client
```
# yum install nfs-utils
# mount -t nfs {nfs-server}:/var/nfs/share /tmp
# ls -la /tmp #可以看到测试文件nfs-test
# umount /tmp #卸载

# exportfs -r #更新配置后的刷新命令
```
##### 4、k8s yaml中使用NFS
```
 volumes:
    - name: gradlehome
      nfs:
        server: nfs-server-ip
        path: "/var/nfs/share"  
```

## 备注：也可以在k8s中创建pv和pvc来管理持久存储