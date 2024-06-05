# ctr常用操作

## 查看镜像

```bash
ctr -n k8s.io image ls 
```

### 删除镜像

```bash
ctr image delete 10.202.43.191:5000/kubernetes/pause/pause:3.6
```

### 以http方式拉取镜像

```bash
ctr image pull --plain-http=true 10.202.43.191:5000/kubernetes/pause/pause:3.6
```





# crictl常用操作

## 查看镜像

```bash
crictl image ls 
```

## 删除镜像

```bash
crictl rmi <image_name:image_tag>
```

