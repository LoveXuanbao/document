## docker 启动gitlab命令

```bash
docker run --detach \
  --publish 443:443 --publish 8880:80 --publish 10022:22 \
  --name gitlab \
  --restart always \
  --volume /data/gitlab/config:/etc/gitlab \
  --volume /data/gitlab/logs:/var/log/gitlab \
  --volume /data/gitlab/data:/var/opt/gitlab \
  --privileged=true \
  gitlab/gitlab-ce:latest
```



## docker 启动gogs

```bash
docker run -d -it --name gogs \
    -p 10022:22 \
    -p 10888:3000 \
    -v /var/gogs:/data/gogs_data \
    repository.test.com:8444/kubernetes/gogs/gogs:0.14.0
```
