

```bash
systemctl stop docker

cp -aR /data/docker_customized /data/sdb1/

mv /data/docker_customized /data/docker_customized_bak

sed -i 's#graph=/data/docker_customized#graph=/data/sdb1/docker_customized#g' /usr/lib/systemd/system/docker.service

systemctl daemon-reload  && systemctl restart docker  && systemctl status docker
```

