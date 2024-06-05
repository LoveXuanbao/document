### /etc/keepalived/keepalived.conf

```bash
# 全局定义，还可以设置发送邮件等功能
global_defs {
    router_id  haproxy-master    # 路由ID，表示本节点的字符串，邮件通知时会用到
    defautl_interface  eth0
}

# 自定义VRRP实例健康检查脚本；keepavlied只能做到对自身问题和网络故障的监控，Script可以增加其他的监控来判定是否需要切换主备
vrrp_script chk_haproxy {
    script "/etc/keepalived/scripts/haproxy_check.sh"    # 心跳检测脚本，检测haproxy是否启动
    interval 2    # 检测脚本执行的间隔，单位是秒
    timeout 2
    fall 3
}

# VRRP实例，定义对外提供服务的VIP区域及其相关属性
vrrp_instance haproxy {
    state MASTER    # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface ech0    # 节点固有IP（非VIP）的网卡，用来发VRRP包
    virtual_router_id 20    # 虚拟路由编号，主从要一致
    priority 150    # 优先级，数值越大，获取处理请求的优先级越高，主从之间最好差50

    # 设置验证类型和密码，MASTER和BACKUP必须使用相同的密码才能正常通信
    authentication {
        auth_type PASS
        auth_pass 20
    }

    # 配置虚拟IP（VIP）
    virtual_ipaddress {
        172.16.104.7     # 定义虚拟IP（VIP）
    }

    # 自定义健康检测脚本
    track_script {
        chk_haproxy    # 配置上面自定义的VRRP脚本调用名
    }

    # 记录切换为主节点的信息
    notify_master "/etc/keepalived/scripts/haproxy_master.sh"
}
```



### &#x20;/etc/keepalived/scripts/haproxy\_check.sh

```bash
#!/bin/bash
LOGFILE="/var/log/keepalived-haproxy-state.log"
date >> $LOGFILE
A=`ps -C haproxy --no-header | wc -l`
if [ $A -eq 0 ]; then
    echo "fail: check_haproxy status" >> $LOGFILE
    echo "Try to start haproxy service" >> $LOGFILE
    systemctl start haproxy    # 重启haproxy服务
    sleep 2
    if [ `ps -C haproxy --no-header | wc -l` -eq 0 ]; then    # 如果启动失败，就进行VIP漂移
        echo "Can not start haproxy service, so will stop keepalived service" >> $LOGFILE
        systemctl stop keepalived
    fi
else
    echo "success: check_haproxy status" >> $LOGFILE
fi

```