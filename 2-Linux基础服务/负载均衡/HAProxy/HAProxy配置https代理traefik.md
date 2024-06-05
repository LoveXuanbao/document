## haproxy代理https域名转发

### 背景

> k8s集群（三台master节点，两台haproxy+keepalived，N台node节点）高可用使用的是haproxy+keepalibed做的高可用负载均衡，k8s上部署的服务通过treafik配置了各个产品的域名（自签证书），虽然可以使用不同的node节点IP作为hosts映射，但是考虑到服务的可用性，如果刚好映射的那台节点意外宕机，hosts解析后而连不通，造成的服务不可用。为了不增加资源，在现有集群基础上做traefik服务的高可用

### haproxy配置 traefik的配置

```bash
global
    daemon
    maxconn 30000
    log 127.0.0.1 local0 debug
    nbproc          2
    tune.ssl.default-dh-param 2048         ## 因为我们的ssl密钥使用的是2048bit加密，所以在次申明，不然又warnning报警

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http          ## 运行模式tcp、http、health
    option                  forwardfor   ## 
    log                     global
    option                  tcplog
    option                  dontlognull
    option http-server-close
    #option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3              ## 三次lia
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 30000

frontend kubernetes-apiserver
    mode                 tcp
    bind                 10.202.83.151:6443
    option               tcplog
    default_backend      kubernetes-apiserver

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode        tcp
    balance     static-rr
    server  master-0 10.202.60.250:6443 check inter 10s rise 3 fall 3
    server  master-1 10.202.61.116:6443 check inter 10s rise 3 fall 3
    server  master-2 10.202.61.221:6443 check inter 10s rise 3 fall 3


listen stats
    bind *:1080
    mode http
    stats refresh 30s
    stats uri /stats

frontend frontend_https
    bind               *:80
    bind               *:443 ssl crt /usr/local/etc/haproxy/soctp.pem
    mode               http
    option             httpclose          ## 每次请求完毕后主动关闭http通道
    #reqadd             X-Forwarded-Proto:\ https
    redirect code 301 scheme  https if !{ ssl_fc }
    acl is_1 hdr_end(host) -i soctp.com
    use_backend backend_https if is_1
    default_backend    backend_https
backend backend_https
    mode      http
    balance   static-rr
    server nodes-1 10.202.60.181:80  check inter 10s rise 3 fall 3
    server nodes-2 10.202.61.107:80  check inter 10s rise 3 fall 3
    server nodes-3 10.202.83.128:80  check inter 10s rise 3 fall 3
```