## Nexus因磁盘空间占满异常停机 orientdb数据损坏修复

- 公司同事发现nexus下载文件异常，报错信息如下

![image-20230825170051337](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170051337.png)

- 登录服务器查看，查看nexus日志，发现磁盘无法写入，使用命令查看系统磁盘占用情况，发现磁盘nexus的数据盘已占用100%

![image-20230825170130439](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170130439.png)



- 此时，只能清理/data目录上不需要得文件，（如历史日志，删除前需要将nexus进程停止，或者存放在该目录下得别的文件），或者对磁盘扩容
- 注意：千万不要手动删除 `sonatype-work` 目录下的任何文件
- 当解决磁盘空间占用100%的问题后，重新启动nexus服务，仍然报错

![image-20230825170222026](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170222026.png)



- 此问题是 `component` 数据库文件被损坏了，一般需要使用 nexus 自带的 console 工具执行修复命令积即可

> 进入数据库

```
java -jar /data/nexus-3.14.0-04/lib/support/nexus-orient-console.jar
```



> 连接数据库

```
CONNECT PLOCAL:/data/sonatype-work/nexus3/db/component admin admin
```

- 有时候在连接数据库这一步会报错，错误内容跟最初的错误很类似，主要错误信息是这段，意思是数据库未被正确关闭，尝试从历史日志恢复，但又恢复失败，此时需要进入到数据库的文件目录，删除 `*.wal`文件，然后重新打开 `console`工具，执行修复，即可解决

![image-20230825170307268](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825170307268.png)



> 修复命令

```
cd /data/sonatype-work/nexus3/db/component/     ## db后面的目录为数据库目录
rm -rf *.wal

java -jar /data/nexus-3.14.0-04/lib/support/nexus-orient-console.jar
CONNECT PLOCAL:/data/sonatype-work/nexus3/db/component admin admin    ## db后面的目录为数据库目录

REBUILD INDEX *
REPAIR DATABASE --fix-graph
REPAIR DATABASE --fix-links
REPAIR DATABASE --fix-ridbags
REPAIR DATABASE --fix-bonsai
DISCONNECT
```

- 如果有多个数据被损坏，重复上面的步骤即可，注意修改数据库文件夹的目录。

- 此时问题基本解决，重启 nexus 服务即可



> 修复过程

```
java -jar /data/nexus-3.14.0-04/lib/support/nexus-orient-console.jar

OrientDB console v.2.2.36 (build d3beb772c02098ceaea89779a7afd4b7305d3788, branch 2.2.x) https://www.orientdb.com
Type 'help' to display all the supported commands.
orientdb> CONNECT PLOCAL:/data/sonatype-work/nexus3/db/component admin admin

Connecting to database [PLOCAL:/data/sonatype-work/nexus3/db/component] with user 'admin'...
2021-07-28 18:11:12:052 WARNI {db=component} Storage 'component' was not closed properly. Will try to recover from write ahead log... [OLocalPaginatedStorage]
2021-07-28 18:11:12:052 WARNI {db=component} Restore is not possible because write ahead log is empty. [OLocalPaginatedStorage]OK
orientdb {db=component}> REBUILD INDEX *


Rebuilding index(es)...
Rebuilt index(es). Found 306689 link(s) in 114.259003 sec(s).


Index(es) rebuilt successfully
orientdb {db=component}> REPAIR DATABASE --fix-graph

Repairing database...
- Removing broken links...
-- Done! Fixed links: 0, modified documents: 0
Repair database complete (0 errors)
orientdb {db=component}>

orientdb {db=component}> REPAIR DATABASE --fix-links

Repairing database...
- Removing broken links...
-- Done! Fixed links: 0, modified documents: 0
Repair database complete (0 errors)
orientdb {db=component}>

orientdb {db=component}> REPAIR DATABASE --fix-ridbags

Repairing database...
- Removing broken links...
-- Done! Fixed links: 0, modified documents: 0
Repair database complete (0 errors)
orientdb {db=component}>

orientdb {db=component}> REPAIR DATABASE --fix-bonsai

Repairing database...
- Removing broken links...
-- Done! Fixed links: 0, modified documents: 0
Repair database complete (0 errors)
orientdb {db=component}>

orientdb {db=component}> DISCONNECT

Disconnecting from the database [component]...OK
orientdb> 
```