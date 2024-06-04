安装FTP

更新系统并安装vsftpd软件包

```bash
yum install vsftpd
```

启动并设置FTP服务器

```bash
systemctl start vsftpd
systemctl enable vsftpd
```



sshd_config

```bash
Subsystem      sftp    /usr/libexec/openssh/sftp-server
```



