### Cloudera Manager Agent could not determine the duplex mode or interface speed.

- 问题原因：Linux服务器有多张网卡，CDH检测网络时采集了 virbr0 的网卡信息，事实上 virbr0 是不可用的；
- 解决办法：

首页 --> Hosts --> All Hosts --> Configuration --> 搜索框输入：Interface

![image-20231212111716586](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231212111716586.png)







![image-20231212153601043](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20231212153601043.png)