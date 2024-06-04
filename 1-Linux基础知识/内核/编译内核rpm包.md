## 内核下同版本下载地址

https://mirrors.tuna.tsinghua.edu.cn/kernel/

## 安装依赖项

```bash
yum install -y ncurses-devel  elfutils-libelf-devel  bc gcc flex bison openssl-devel rpm-build
```

```bash
gcc* ncurses-devel flex bison rpm-build bc
```

## 解压官网下载的tar.xz包

```bash
tar -xf linux-5.10.166.tar.xz
cd linux-5.10.166
make menuconfig
```

![image-20230825115126828](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825115126828.png)