```
[root@repository nexus]# pwd
/data/nexus
[root@repository nexus]# ll
total 0
drwxr-xr-x. 9 nexus nexus 188 Oct 21 20:34 nexus-3.14.0-04
drwxr-xr-x. 4 nexus nexus  38 Jan  7  2021 sonatype-work

systemctl stop nexus

[root@repository nexus]# cd nexus-3.14.0-04/
[root@repository nexus-3.14.0-04]# java -jar ./lib/support/nexus-orient-console.jar

connect plocal:../sonatype-work/nexus3/db/security admin admin

update user SET password="$shiro1$SHA-512$1024$NE+wqQq/TmjZMvfI7ENh/g==$V4yPw8T64UQ6GfJfxYq2hLsVrBY8D1v+bktfOxGdt4b/9BthpWPNUy/CBk6V9iA0nHpzYzJFWO8v/tZFtES8CA==" UPSERT WHERE id="admin"

systemctl start nexus
```