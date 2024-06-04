Kdump是在系统崩溃、死锁或死机时用来记录内存运行参数的一个工具和服务，是一种新的crash dump捕获机制，用来捕获内核崩溃的时候产生的crash dump。kdump需要配置两个不同目的的内核，其中一个我们在这里称作标准的内核；另一个称之为捕捉内核。打个比方，如果系统一旦崩溃，那么正常的内核就没有办法工作了，在这个时候将由kdump产生一个用于捕获当前运行信息的内核，该内核会将此时的内存中的所有运行状态和数据信息收集到一个dump core文件中以便于分析崩溃的原因，一旦内存信息收集完成，系统将会自动重启


kdump机制主要包括两个组件：kdump和kexec

kexec是一个快速启动kernel的机制，它运行在某一正在运行的内核中，启动一个新的内核，而且不用重新经过BIOS就可以完成启动。因为一般BIOS都会花费很长的时间，尤其是在大型并且同时连接许多外部设备的的环境下，BIOS会花费更多的时间。在生产内核崩溃之后运行捕捉内核。
kexec是kdump机制的关键，包含两部分：
内核空间的系统调用kexec_load。负责在生产内核启动时将捕获内核加载到指定地址。
用户空间的工具kexec-tools。将捕获内核的地址传递给生产内核，从而在系统崩溃的时候找到捕获内核的地址并运行。



安装 kdump 服务

首先查看系统内核版本

![image-20240307180333690](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180333690.png)


根据自己的版本到此处  http://debuginfo.centos.org/7/x86_64/  下载 debug_info  内核调试工具

![image-20240307180359267](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180359267.png)

![image-20240307180423361](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180423361.png)

安装  内核调试工具

rpm -ivh kernel-debuginfo-common-x86_64-3.10.0-514.el7.x86_64.rpm

rpm -ivh kernel-debuginfo-3.10.0-514.el7.x86_64.rpm

安装  crash  和  kexec-tools

rpm -qa | grep crash

rpm -qa | grep kexec-tools

![image-20240307180449601](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180449601.png)

安装 crash

yum install crash -y

kdump 服务器在启动前 必须在内存中预留了 空间  如果没有 则无法启动

会提示：
![image-20240307180534403](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180534403.png)

查看是否已预留了空间
查看预留内存
cat /sys/kernel/kexec_crash_size 

![image-20240307180626931](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180626931.png)

如果是0  则没有预留

预留的方法：

vim  /etc/default/grub

在这一行中添加 crashkernel=0M-2G:0M,2G-8G:192M,8G-:256M
意思是 物理内存 2G内 不预留  8G以内 预留192M  8G以上预留256M

![image-20240307180650424](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180650424.png)

修改好后  要重新生成 grub 文件

！！！！！！！这一步操作后 需要重启服务器后 才可以生效！！！！！！！！！

grub2-mkconfig -o /boot/grub2/grub.cfg

![image-20240307180711294](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180711294.png)

安装了 kernel-debuginfo   crash  kexec-tools 后就可以启动 kdump服务了

kdump配置文件位置：

![image-20240307180728365](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180728365.png)

一般修改这两项即可

vmcore文件产生后 存储位置

生成 vmcore 文件的命令

![image-20240307180753395](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180753395.png)


![image-20240307180811416](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180811416.png)

模拟内核崩溃：  此操作会导致服务器重启 ！！！！！！！！！！

echo  c  > /proc/sysrq-trigger

执行后 终端的连接会断开  等重启后  登录后会在 kdump 配置文件中指定位置 产生以系统崩溃时间命名的目录

进入后会看到  vmcore  文件

![image-20240307180828972](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180828972.png)

![image-20240307180847783](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180847783.png)


分析 vmcore 文件

命令：

crash /usr/lib/debug/lib/modules/3.10.0-514.el7.x86_64/vmlinux /var/crash/127.0.0.1-2021-11-11-11\:52\:55/vmcore

![image-20240307180907158](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180907158.png)


crash  是以交互的形式来运行的 

最开始产生的信息：

![image-20240307180927513](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307180927513.png)

- **KERNEL**：指明 crash 时运行的 kernel；
- **DUMPFILE**：转储的内存 core 文件名称；
- **CPUS**：机器上 CPU 个数；
- **DATE**：crash 发生的时间；
- **TASKS**：crash 时内存中运行任务的数量；
- **NODENAME**：crash 的主机名;
- **RELEASE and VERSION**：kernel 的版本号；
- **MACHINE**：机器的 CPU 架构；
- **MEMORY**：机器的内存大小；
- **PANIC**：系统 panic 类型，如： Oops、soft、lockup、hard lockup oom、hung task等；
- **PID**：触发 panic 的进程 ID；
- **COMMAND**：触发 panic 的进程名称；
- **TASK**：问题进程地址；
- **CPU**：crash 时进程运行的当前 CPU 号；
- **STATE**：crash 时进程的运行状态；

解析：
    产生Panic的原因是sysrq触发了一个crash（Ps：我使用了sysrq模拟内核崩溃“echo c > /proc/sysrq-trigger”）


bt命令
通过“bt”命令+上述的“PID”字段，可以打印问题进程的栈信息，如下：

![image-20240307181746166](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307181746166.png)

sym命令

“sym 内存地址”可以看到这个地址上对应的符号表信息，并且具体到源代码的那一行！！！

crash> sym ffffffff813baf16

![image-20240307181802322](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307181802322.png)


查看次文件  可以找到问题的原因

![image-20240307181816668](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20240307181816668.png)