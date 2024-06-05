```
[root@gs-server-12122 certs]# openssl req -newkey rsa:2048 -nodes -keyout tls.key -x509 -days 3650 -out tls.crt
Generating a 2048 bit RSA private key
................+++
.................................................................................................................+++
writing new private key to 'tls.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN      ## 国家
State or Province Name (full name) []:BeiJing     ## 省份
Locality Name (eg, city) [Default City]:BeiJing    ## 城市 
Organization Name (eg, company) [Default Company Ltd]:GridSum     ## 公司全称
Organizational Unit Name (eg, section) []:GridSum    ## 公司名称
Common Name (eg, your name or your server's hostname) []:soctp    ## 项目名字
Email Address []:soctp@test.com   ## 测试邮箱
```

```
[root@gs-server-12122 certs]# kubectl create secret generic traefik-cert --from-file=tls.crt --from-file=tls.key -n kube-system
secret/traefik-cert created
```

## 修改 traefik的 `daemonset`
```
    spec:
      volumes:
        ...  
        其他内容
        - name: ssl
          secret:
            secretName: traefik-cert
            defaultMode: 420
        ...
        ...
        其他内容
          volumeMounts:
            ...
            其他内容
            - name: ssl
              mountPath: /ssl
```

## 修改traefik的configmap
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: traefik-conf
  namespace: kube-system
  selfLink: /api/v1/namespaces/kube-system/configmaps/traefik-conf
  uid: e888dc98-a6b4-4ac2-b369-6c853114876b
  resourceVersion: '892'
  creationTimestamp: '2022-07-05T10:38:56Z'
data:
  config.toml: |
    defaultEntryPoints = ["http","https"]
    insecureSkipVerify = true
    [entryPoints]
      [entryPoints.http]
      address = ":80"
      [entryPoints.https]
      address = ":443"
        [entryPoints.https.tls]
           minVersion = "VersionTLS12"
           cipherSuites = [
             "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384",
             "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
             "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
             "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
             "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
             "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
             "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
             "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
           ]
          ### 新增内容
          [[entryPoints.https.tls.certificates]]
          certFile = "/ssl/tls.crt"
          keyFile = "/ssl/tls.key"
binaryData: {}
```