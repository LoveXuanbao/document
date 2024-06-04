## awk截取过滤大于指定值的方法

重启`moebius-system`命名空间下，RESTART次数大于0的所有pod

```
kubectl get pod -n moebius-system | awk '$4>0{print $1}'  | xargs kubectl delete pod -n moebius-system
```

`awk '$4>0{print $1}'`命令详解

打印第一列，并且第四列的值大于0的行



## awk截图端口和服务名称
```
netstat -tnpl|awk 'NF==7{sub(/.*:/,"",$4);sub(/[0-9]*\//,"",$NF);print $4,$NF}'
 
netstat -tnpl|awk '/^tcp/{sub(/.*:/,"",$4);sub(/[0-9]*\//,"",$7);print $4,$7}'
 
netstat -tnpl|awk '/^tcp/{sub(/.*:/,"",$4);sub(/[0-9]*\//,"",$7);if(NF==7)print $4,$7;else print $4,$7,$NF}'
```


## awk 截取服务器IP地址
```
ifconfig ens160 | awk 'NR==2{print $2}'
```

## 截取除最后一列的其他列

```bash
grep -r 'image:' $i | awk '{print $2}' | awk -F/ '{$NF=""; print $0}' | sed 's# #/#g'
```



# 删除匹配行到最后一行

```bash
sed '/^status/,$d' eg-app-svc.yaml 
```

## 按小时分割文件

```bash
awk '{split($4, a, ":"); print >> "access.log."a[2]}' access.log
```

