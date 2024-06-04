```

[nexus]
name=Nexus Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/os/$basearch/
enabled=1
gpgcheck=0

[updates]
name=updates Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/updates/$basearch/
enabled=1
gpgcheck=0

[extras]
name=extras Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/extras/$basearch/
enabled=1
gpgcheck=0

[centosplus]
name=centosplus Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/centosplus/$basearch/
enabled=1
gpgcheck=0

[epel]
name=epel Repository
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum/$releasever/$basearch/
enabled=1
gpgcheck=0


[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=http://admin:admin123@192.168.88.100:8081/repository/yum//$releasever/$basearch/stable
enabled=1
gpgcheck=0

```



```
[nexus]
name=Nexus Repository
baseurl=http://admin:admin123@localhost:8081/repository/yum/centos/$releasever
enabled=1
gpgcheck=0
```