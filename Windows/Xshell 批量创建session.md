批量添加的首要条件是：要保证所有主机的用户名和密码是一致的

### 原理

复制一个`session`连接文件，修改文件中的`IP`为要添加的`seesion`地址。

### 实现过程

1. 现在`xshell`里，手动添加一台主机，然后找到这个`seesion`文件

2. 把这个文件拷贝到一台`Linux`主机上的任意目录 。注意：这个目录也是我们下面脚本的存放目录。

3. `session`保存的名称为：用途_IP地址的后两端，如：`IP`地址为`192.168.10.100`，保证的则为 `ansible_10.100`

4. `IP`地址列表格式如下：

   ```
   ansible         192.168.10.100
   haproxy-1       192.168.10.101
   haproxy-2       192.168.10.102
   nexus           192.168.10.103
   patroni-1       192.168.10.104
   patroni-2       192.168.10.105
   patroni-3       192.168.10.106
   sso-1           192.168.10.107
   sso-2           192.168.10.108
   tenant-1        192.168.10.109
   tenant-2        192.168.10.110
   panel-1         192.168.10.111
   panel-2         192.168.10.112
   ....
   ```

5. 批量创建脚本如下：脚本执行完成之后，可直接把`session`存放目录直接放入到`xshell`存放`session`目录中

   ```shell
   #!/bin/bash
   
   #因为我们for循环里cat的文件每一行 "ansible        10.1.1.1" 类型的，在每一行中间有空格，echo输出的是时候会把空格认定为换行符
   IFS=$'\n'
   
   for LINE in `cat ./ip.txt`; do
     HOSTS=`echo $LINE | awk '{print $1}'`
     IP=`echo $LINE | awk '{print $2}'`
     NUM=`echo $IP | awk -F. '{print $4}'`
     cp ansibl_97.35.xsh ./test/"$HOSTS"_97."$NUM.xsh"
     sed -i "s/^Host.*/Host=$IP/" ./test/"$HOSTS"_97."$NUM.xsh"
   done
   
   #创建session文件夹，并把session移动到对应的文件夹下
   mkdir -p ./test/{Ansible,Haproxy,Nexus,Patroni,AgileBI,Datascience,Moebius,SSO,tenant,ZETA}
   mv ./test/ansible_* ./test/Ansible/
   mv ./test/haproxy-* ./test/Haproxy/
   mv ./test/nexus_* ./test/Nexus/
   mv ./test/patroni-* ./test/Patroni/
   mv ./test/agilebi-* ./test/AgileBI/
   mv ./test/datascience-* ./test/Datascience/
   mv ./test/k8s* ./test/Moebius/
   mv ./test/sso-* ./test/SSO/
   mv ./test/tenant-* ./test/tenant/
   mv ./test/panel* ./test/ZETA/
   mv ./test/zeta* ./test/ZETA/
   ```

   