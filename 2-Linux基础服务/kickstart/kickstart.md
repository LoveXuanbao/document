# kickstart自动批量安装操作系统

PEX（网络启动）工作示意：

![image-20230825145139651](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825145139651.png)

**PEX客户端：**没有装系统的新机器

**DHCP服务器/TFTP服务器：**是kickstart服务需要的两个基本服务DHCP和TFTP，DCHP服务可以和kickstart服务器不在同一台主机上

**DHCP服务器：**为了给PEX客户端分配IP地址，客户端获取到IP之后，就要去找安装系统所需要的文件存放位置

**TFTP服务器：**是一种非交互式的共享文件服务，用户共享PXE客户端安装系统时所需要用到的一些文件，让客户端以非交互式的方式来下载这些文件



# kickstart自动批量部署案例

