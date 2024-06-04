# 麒麟v10 arm 版安装mysql

## 服务器信息

```bash
[root@localhost ~]# cat /etc/os-release 
NAME="Kylin Linux Advanced Server"
VERSION="V10 (Tercel)"
ID="kylin"
VERSION_ID="V10"
PRETTY_NAME="Kylin Linux Advanced Server V10 (Tercel)"
ANSI_COLOR="0;31"


[root@localhost ~]# cat /proc/version 
Linux version 4.19.90-23.6.v2101.ky10.aarch64 (KYLINSOFT@localhost.localdomain) (gcc version 7.3.0 (GCC)) #1 SMP Wed Mar 17 14:45:17 CST 2021


[root@localhost ~]# hostnamectl 
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: b43e4902d3214f7d97d6cd5ecf0ea8d1
           Boot ID: 1db80bae76ac4b289566a3038358d79e
    Virtualization: kvm
  Operating System: Kylin Linux Advanced Server V10 (Tercel)
            Kernel: Linux 4.19.90-23.6.v2101.ky10.aarch64
      Architecture: arm64
```

## 访问mysql官方，下载arm版

- 下载地址，注意选择版本为 ARM的，官网只有8.0.31版本之后有ARM版

```bash
https://downloads.mysql.com/archives/community/
```

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230506160417.png)

## 上传下载好的安装包到服务器

- 在服务器/tmp目录下创建mysql目录

```bash
mkdir /tmp/mysql
```

- 上传安装包到/tmp/mysql目录下

```bash
[root@localhost mysql]# pwd
/tmp/mysql
[root@localhost mysql]# ll
-rw-r--r-- 1 root root 511786827  9月 14  2022 mysql-8.0.31-linux-glibc2.17-aarch64.tar.gz
```

## 安装mysql

- 创建mysql用户和mysql组

```bash
[root@localhost ~]# groupadd mysql
[root@localhost ~]# useradd -g mysql mysql
```

- /usr/local 目录下创建mysql目录

```bash
[root@localhost ~]# mkdir /usr/local/mysql
```

- 解压安装包

```bash
[root@localhost ~]# cd /tmp/mysql
[root@localhost mysql]# tar -zxf mysql-8.0.31-linux-glibc2.17-aarch64.tar.gz 
[root@localhost mysql]# ll
总用量 499840
drwxr-xr-x 9 root root       220  5月  6 16:04 mysql-8.0.31-linux-glibc2.17-aarch64
-rw-r--r-- 1 root root 511786827  9月 14  2022 mysql-8.0.31-linux-glibc2.17-aarch64.tar.gz
```

- 拷贝解压目录下的内容到/usr/local/mysql目录下

```bash
[root@localhost mysql]# mv mysql-8.0.31-linux-glibc2.17-aarch64/* /usr/local/mysql/
```

- 修改/usr/local/mysql目录权限

```bash
[root@localhost mysql]# chown mysql.mysql -R /usr/local/mysql
[root@localhost mysql]# ll /usr/local/mysql/
总用量 296
drwxr-xr-x  2 mysql mysql   4096  9月 14  2022 bin
drwxr-xr-x  2 mysql mysql     55  9月 14  2022 docs
drwxr-xr-x  3 mysql mysql    282  9月 14  2022 include
drwxr-xr-x  6 mysql mysql    201  9月 14  2022 lib
-rw-r--r--  1 mysql mysql 287627  9月 14  2022 LICENSE
drwxr-xr-x  4 mysql mysql     30  9月 14  2022 man
-rw-r--r--  1 mysql mysql    666  9月 14  2022 README
drwxr-xr-x 28 mysql mysql   4096  9月 14  2022 share
drwxr-xr-x  2 mysql mysql     77  9月 14  2022 support-files
```

- 数据盘创建mysql数据目录和日志目录并赋予mysql权限

```bash
[root@localhost mysql]# mkdir -p /data/mysql/{data, logs, binlog}
[root@localhost mysql]# chown -R mysql.mysql /data/mysql
```

- /etc/profile添加环境变量

```bash
[root@localhost mysql]# vim /etc/profile
...
# 最后一行添加如下内容
export PATH=$PATH:/usr/local/mysql/bin

[root@localhost mysql]# source /etc/profile    # 使配置生效
```

- 使用mysql -V 验证是否能查看mysql版本信息

```bash
[root@localhost mysql]# mysql -V
mysql  Ver 8.0.31 for Linux on aarch64 (MySQL Community Server - GPL)
```

- 编写my.cnf文件

```bash
[root@localhost mysql]# vim /etc/my.cnf
[mysqld]
   basedir=/usr/local/mysql
   bind-address=0.0.0.0
   datadir=/data/mysql/data
   log-error=/data/mysql/logs/mysql.err
   lower-case-table-names=1
   pid-file=/data/mysql/mysql.pid
   port=3306
   server_id=1
   socket=/data/mysql/binlog/mysql.sock
   user=mysql
   #character config
   character_set_server=utf8mb4
   symbolic-links=0
[mysql]
socket=/data/mysql/binlog/mysql.sock
```

- 格式化数据库

```bash
[root@localhost mysql]# mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data
```

- 查看mysql初始密码

  文件最后的 `root@localhost:` 后面的跟着的就是mysql的初始密码

```bash
[root@localhost mysql]# cd /data/mysql/logs/
[root@localhost logs]# ll
总用量 4
-rw-r----- 1 mysql mysql 1777  5月  6 15:25 mysql.err
[root@localhost logs]# tail -f mysql.err 
2023-05-06T07:23:23.889866Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.31) initializing of server in progress as process 24953
2023-05-06T07:23:23.889928Z 0 [ERROR] [MY-010338] [Server] Can't find error-message file '/data/mysql/mysql-8.0.31-linux-glibc2.17-aarch64/share/errmsg.sys'. Check error-message file location and 'lc-messages-dir' configuration directive.
2023-05-06T07:23:23.891881Z 0 [ERROR] [MY-013236] [Server] The designated data directory /data/mysql/mysql-8.0.31-linux-glibc2.17-aarch64/data/ is unusable. You can remove all files that the server added to it.
2023-05-06T07:23:23.892004Z 0 [ERROR] [MY-010119] [Server] Aborting
2023-05-06T07:23:23.892216Z 0 [System] [MY-010910] [Server] /usr/local/mysql/bin/mysqld: Shutdown complete (mysqld 8.0.31)  MySQL Community Server - GPL.
2023-05-06T07:25:10.605381Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2023-05-06T07:25:10.605550Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.31) initializing of server in progress as process 24964
2023-05-06T07:25:10.619408Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2023-05-06T07:25:10.929239Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2023-05-06T07:25:13.991723Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: wu=_:6Gpnfh-
```

- 准备启动脚本

```bash
[root@localhost mysql]# cd /usr/local/mysql/support-files/
[root@localhost support-files]# cp mysql.server /etc/init.d/mysql
```

- mysql启动停止重启命令

```bash
[root@localhost support-files]# service mysql status     ## 查看mysql服务状态
[root@localhost support-files]# service mysql start      ## 启动mysql
[root@localhost support-files]# service mysql stop       ## 停止mysql
[root@localhost support-files]# service mysql restart    ## 重启mysql
```

- mysql修改初始密码，并开启root用户远程访问数据库权限

```bash
# mysql启动后，使用  mysql -uroot -p 命令进入数据库

[root@localhost support-files]# mysql -uroot -p
Enter password:     # 此处输入logs文件中生成的初始密码
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.31

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user 'root'@'localhost' identified by 'gridsum@2023!';    # 使用alter语句修改root用户密码
Query OK, 0 rows affected (0.01 sec)


# 开启root用户远程访问数据库权限
mysql> use mysql;   # 切换到mysql数据
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select User,authentication_string,Host from user;    # 查看用户权限，可以发现root用户的host只允许主机访问
+------------------+------------------------------------------------------------------------+-----------+
| User             | authentication_string                                                  | Host      |
+------------------+------------------------------------------------------------------------+-----------+
| mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
| mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
| mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
| root             | $A$005$}|M4Id8)<!d( CqEea/DV/aSTrgDZA7/kQVqMFzjesvMANowwRfpJYg4j/1o.s2 | localhost |
+------------------+------------------------------------------------------------------------+-----------+
4 rows in set (0.00 sec)


mysql> update user set host='%' where user='root';     # 修改允许所有主机通过root用户访问
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> 
mysql> 
mysql> 
mysql> select User,authentication_string,Host from user;
+------------------+------------------------------------------------------------------------+-----------+
| User             | authentication_string                                                  | Host      |
+------------------+------------------------------------------------------------------------+-----------+
| root             | $A$005$}|M4Id8)<!d( CqEea/DV/aSTrgDZA7/kQVqMFzjesvMANowwRfpJYg4j/1o.s2 | %         |
| mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
| mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
| mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | localhost |
+------------------+------------------------------------------------------------------------+-----------+


mysql> flush privileges;   # 刷新缓存
Query OK, 0 rows affected (0.01 sec)
```

## 使用navicat客户端工具连接

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230506163310.png)