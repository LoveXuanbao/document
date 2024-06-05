```bash
global
    log 127.0.0.1 local3 info   # 在本机记录日志
    chroot   /var/lib/haproxy   # haproxy安装目录
    pidfile  /var/run/haproxy.pid
    maxconn  20000   # 每个进程可用的最大连接数
    user    haproxy
    group   haproxy
    daemon   # 以后台守护进行运行

defaults
    log global
    mode http   # 运行模式 tcp、http、health
    retries 3   # 三次连接失败，则判断服务不可用
    option redispatch   # 如果后端有服务器宕机，强制切换到正常服务器
    stats uri /haproxy   # 统计页面URL路径
    stats refresh 30s   # 统计页面自动刷新时间
    stats realm haproxy-status   # 统计页面输入密码框提示信息
    stats auth admin:admin   # 统计页面用户名和密码
    stats hide-version   # 隐藏统计页面上haproxy版本信息
    maxconn 65536   # 每个进程可用的最大连接数
    timeout http request   # 在客户端建立连接但不请求数据时，关闭客户端连接
    timeout queue   # 等待最大时长
    timeout connect   # 定义haproxy将客户端请求转发至后端服务器所等待的超时时长
    timeout client   # 客户端非活动状态的超时时长
    timeout server   # 客户端与服务器端建立连接后，等待服务器端的超时时长
    timeout http-keep-alive   # 定义保持连接的超时时长
    timeout check   # 健康状态检测时的超时时间，过短会误判，过长会浪费资源
```