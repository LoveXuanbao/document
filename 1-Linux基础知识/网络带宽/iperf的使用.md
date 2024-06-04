# 安装
```
手动安装 iperf3 3.1.3 64 位：
sudo wget -O /usr/lib/libiperf.so.0 https://iperf.fr/download/ubuntu/libiperf.so.0_3.1.3
sudo wget -O /usr/bin/iperf3 https://iperf.fr/download/ubuntu/iperf3_3.1.3
sudo chmod +x /usr/bin/iperf3
```


## 服务端
```
iperf3 -s
```


## 客户端
```
cd /d D:\personal data\带宽测试\iperf-3.1.3-win64\iperf-3.1.3-win64 ## 切换目录
iperf3.exe -c xx.xx.xx.xx
```