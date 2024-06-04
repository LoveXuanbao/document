## cfssl生成证书
###  先下载cfssl工具
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssljson
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/bin/cfsslcertinfo

chmod +x /usr/bin/cfssl*

ll /usr/bin/cfssl*
-rwxr-xr-x 1 root root 10376657 Mar 30  2016 /usr/bin/cfssl
-rwxr-xr-x 1 root root  6595195 Mar 30  2016 /usr/bin/cfssl-certinfo
-rwxr-xr-x 1 root root  2277873 Mar 30  2016 /usr/bin/cfssl-json
```

###  创建 soctp-ca-csr.json
```
{
  "CN": "soctp",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Beijing",
      "L": "Beijing",
      "O": "etcd",
      "OU": "Etcd Security"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}
```

###  执行下面命令生成根证书
```
cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /opt/ssl/soctp-ca
# 然后会在/opt/ssl目录下生产三个文件
soctp-ca.csr  soctp-ca-key.pem  soctp-ca.pem
```

###  用根证书，生成域名证书
#### 先创建 ca-config.json 文件
```
{
    "signing": {
        "default": {
            "expiry": "175200h"
        },
        "profiles": {
            "server": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```
### 执行下面命令生成域名证书
- 会生成三个文件：soctp.csr 	soctp-key.pem  soctp.pem
```
cfssl gencert \
   -ca=/opt/ssl/soctp-ca.pem \
   -ca-key=/opt/ssl/soctp-ca-key.pem \
   -config=ca-config.json \
   -profile=server \
   soctp-csr.json | cfssljson -bare /opt/ssl/soctp
```

## 创建traefik的secret
```
[root@gs-server-12122 certs]# kubectl create secret generic traefik-cert --from-file=soctp.pem --from-file=soctp-key.pem -n kube-system
secret/traefik-cert created
```