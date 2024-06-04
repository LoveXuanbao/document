## 准备工作
1、 拿到申请域名时候的ca根证书，或者从浏览器导出证书为`xxx.cer`文件



## 添加
1、进入到$JAVA_HOME目录下的lib/security目录
2、执行下面命令
```
keytool -import -alias abc -keystore cacerts -file xxx.cer         ## ## abc为添加时候指定的别名，可以自行指定


Enter keystore password:       ## 按照提示输入密码，官方默认密码为   changeit

Trust this certificate? [no]:  y    ## 按照提示输入  y
Certificate was added to keystore    ## 显示这个证明添加成功
```

## 查看
```
keytool -list -keystore cacerts -alias abc      ## abc为添加时候指定的别名，可以自行指定
```