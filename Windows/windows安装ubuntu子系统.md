## 一、按键盘上windows图标键，点击设置
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014172630.png)

## 二、点击更新和安全
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014172802.png)

## 三、选择开发者选项，把从任意源安装应用按钮打开，按照提示继续
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014172854.png)

## 四、然后返回，点击应用
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014173047.png)

## 五、选择程序和功能
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014173109.png)  

## 六、选择启动或关闭windows功能
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014173140.png)

## 七、勾选使用与Linux的windows子系统
> 点击确定，等待安装完成，选择立即重启启动  

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014173305.png)

## 八、打开cmd或者PowerShell，输入`wsl -l -o`，查看支持的Linux发行版版本
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014174338.png)

## 九、打开windows的Microsoft Store
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014174609.png)

## 十、在搜索框输入`ubuntu`，然后选择相应的版本点击获取安装
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014174647.png)

## 十一、点击开始菜单栏，刚才添加的ubuntu程序，然后等待程序安装
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014174926.png)
![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014174832.png)

## 十二、安装完成之后，按照提示输入用户名密码
> 此处第一次输入，是让新建用户，输入完之后，即可进入到我们安装的ubuntu系统中  

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20221014175309.png)

## 十三、升级wsl1为wsl2

### 1、查看WSL版本号

打开cmd或者PowerShell，执行命令查看wsl版本号

```shell
wsl -l -v
```

可以看到VERSION列显示得2，因为我本地已经升级过，如果显示得是1，请按照一下步骤升级

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230322140953.png)

### 2、启用虚拟机功能

如果是按照本文档安装得子系统，可以省略该步骤

#### 2.1、查看windows主机是否开启虚拟化功能

鼠标移动到屏幕最下方任务栏，右键，点击任务管理器 --> 性能 --> CPU；查看CPU中得虚拟化是否已开启，如果没有请自行百度查看对应机器型号怎么在BIOS中开启虚拟机

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230322141853.png)

### 2.2、查看windows主机是否开启虚拟机功能

#### 按照上面第六步查看 

### 2.3、打开cmd或者PowerShell，执行以下命令在系统中启动虚拟机功能

```shell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```



### 3、下载Linux内核更新包

1. 根据系统类型进行选择 x86架构或者 arm结构得更新包

```bash
x64:    https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
arm64：https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi
```

2. 下载完毕后，双击运行安装即可

### 4、设置分发版版本

1. 打开cmd或者PowerShell，执行命令查看wsl版本号

```shell
wsl -l -v

  NAME            STATE           VERSION
* Ubuntu-20.04    Running         1
```

2. 将Ubuntu-20.04 设置为WSL2

   该命令会执行一段时间，如果安装成功，会出现转移成功字样

```shell
wsl --set-version Ubuntu-20.04 2
```

### 5、升级完wsl后需要做得设置

wsl升级完之后，打开ubuntu子系统的时候，会出现的报错：参考的对象类型不支持尝试的操作

#### 解决办法

在gitub中微软的WSL仓库中已经给出了解决办法：(https://github.com/microsoft/WSL/issues/4177)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230322143401.png)

- 工具下载地址：

```
www.proxifier.com/tmp/Test20200228/NoLsp.exe
```

- 下载完毕后，通过cmd或者PowerShell执行，注意执行  **.\NoLsp.exe**  需要在文件存放的目录下执行，比如我是下载到桌面，就需要先cd 到桌面，然后在执行

```
cd C:\Users\T480\Desktop
.\NoLsp.exe c:\windows\system32\wsl.exe
```

