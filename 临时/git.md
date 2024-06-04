## 本地和远程不同步，并拉取报错

**报错信息**

```bash
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 8 (delta 3), reused 8 (delta 3), pack-reused 0
Unpacking objects: 100% (8/8), 732 bytes | 2.00 KiB/s, done.
From github.com:LoveXuanbao/ansibleCode
 * branch            main       -> FETCH_HEAD
   61fc29e..a506ef5  main       -> origin/main
error: Your local changes to the following files would be overwritten by merge:
        kubernetes_v1.25/roles/host-initialize/tasks/kubernets_host_init.yml
Please commit your changes or stash them before you merge.
Aborting
Updating 61fc29e..a506ef5
```

- 当使用 `git pull` 命令时，Git 会从远程仓库拉取最新的代码，并将其合并到当前分支。如果想要强制拉取远程仓库的所有内容，可以使用下面命令来将本地分支重置为远程分支的最新状态

```bash
git fetch --all 
git reset --hard origin/<branch>
```

