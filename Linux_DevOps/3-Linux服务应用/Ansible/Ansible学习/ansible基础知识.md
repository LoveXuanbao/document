## 一、Ansible 介绍

`ansible`下载地址：https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/

`Ansible`是一个配置管理系统`configuration management system`，`python` 语言是运维人员必须会的语言，`ansible` 是一个基于`python` 开发的自动化运维工具，其功能实现基于`ssh`远程连接服务，`ansible` 可以实现批量系统配置，批量软件部署，批量文件拷贝，批量运行命令等功能, 除了`ansible`之外，还有`saltstack` 等批量管理软件。

### Ansible能做什么？

`ansible`可以帮助运维人员完成一些批量任务，或者完成一些需要经常重复的工作。

比如：同时在 100 台服务器上安装 `Nginx` 服务，并在安装后启动服务。

比如：将某个文件一次性拷贝到 100 台服务器上。

比如：每当有新服务器加入工作环境中，运维人员都要为新服务器部署某个服务，也就是说运维人员需要经常重复的完成相同的工作。

这些场景中运维人员都可以使用到`Ansible`。

### Ansible 软件特点？

1. `ansible`不需要单独安装客户端，`SSH`相当于`ansible`客户端
2. `ansible`不需要启动任何服务，仅需安装对应工具即可。
3. `ansible`依赖大量的python模块来实现批量管理。
4. `ansible`配置文件`/etc/ansible/ansible.cfg`。
5. `ansible`是一种`agentless`(基于SSH)，可实现批量配置、命令执行和控制，基于`Python`实现的自动化运维工具

### Ansible 的特性

- <font color=FF0000>模块化</font>：通过调用相关模块，完成指定任务，切支持任何语言编写的自定义模块；
- <font color=FF0000>剧本 </font>：playbook，可根据需要一次执行完剧本中的所有任务或某些任务；

### Ansible 的架构 

![image-20230825153451462](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825153451462.png)

- 连接插件`conntctior plugins`用于连接主机，用来连接被管理端。

- 核心模块`(core modules)` 连接主机实现操作， 它依赖于具体的模块来做具体的事情。
-  自定义模块`(custom modules)` 根据自己的需求编写具体的模块。
- 插件`(plugins)` 完成模块功能的补充。
- 剧本`(playbooks)` `ansible`的配置文件,将多个任务定义在剧本中，由ansible自动执行。
- 主机清单`(host inventory)`定义`ansible`需要操作主机的范围。

<font color=FF0000>最重要的一点</font>: `ansible`是模块化的, 它所有的操作都依赖于模块, 不需要单独安装客户端`no agents`，基于系统自带的`sshd`服务，`sshd`就相当于`ansible`的客户端, 不需要服务端`no sever`,需要依靠大量的模块实现批量管理, 配置文件 `/etc/ansible/ansible.cfg` (前期不用配置)。

## 二、Ansible 命令使用

- 语法格式

```bash
ansible <pattern_goes_here> -m <module_name> -a <argument>

ansible 匹配模式 -m 模块 -a '需要执行的内容'
```

- 解释说明

匹配模式：即哪些机器生效（可以是某一台，或某一组，或全部），默认模块为`command`，执行常规的`shell`命令

- 选项

```bash
-m name | --module-name=name     ## 指定执行使用的模块
-u username | --user=username     ## 指定远程主机以username运行命令
-s | --sudo     ## 相当于Linux系统下的sudo命令
-usudo_username | --sudo-user=sudo_username     ## 使用sudo，相当于Linux系统下的sudo命令
-C | --check      ## 只检查不实际执行
-e     ## 即extra_vars，引用外部参数
-i     ## 即inventory，指定仓库列表，默认/etc/ansible/hosts
--list-host     ## 列出执行主机列
```

## 三、Ansible 常用模块

```bash
ping模块     ## 检查指定节点机器是否还能连通，用法很简单，不涉及参数，主机如果在线，则回复pong。
raw模块     ## 执行原始的命令，而不是通过模块子系统。
yum模块     ## RedHat和CentOS的软件包安装和管理工具。
apt模式     ## Ubuntu/Debian的软件包安装和管理工具。
pip模块     ## 用于管理Python库依赖项，为了使用pip模块，必须提供参数name或requirements。
synchronize模块     ## 使用rsync同步文件，将主控目录推送到指定节点的目录下。
template模块     ## 基于模板方式生成一个文件复制到远程主机(template使用jinjia2格式作为文件模板，进行文档内变量的替换的模块)
copy模块     ## 在远程主机执行复制操作文件。
user模块     ## user模块请求的是useradd、userdel、usermod三个指令。
group模块     ## group模块请求的是groupadd、groupdel、groupmod三个指令。
service模块     ## 用户管理远程主机的服务
get_url模块     ## 该模块主要用于从http、ftp、https服务器上下载文件(类似于wget)
fetch模块     ## 它用于从远程机器获取文件，并将其本地存储在有主机名组织的文件树中
file模块     ## 主要用于远程主机上的文件操作
lineinfile模块     ## 远程主机上的文件编辑模块
unarchive模块     ## 用户解压文件
command模块和shell模块     ## 用户在各被管理节点运行指定的命令，shell和command的区别: shell模块可以特殊字符，而command是不支持的
hostname模块     ## 修改远程主机名的模块
script模块     ## 在远程主机上执行主控端的脚本，相当于scp+shell组合
stat模块     ## 获取远程主机的状态信息，包括atime、ctime、mtime、md5、uid、gid等信息
cron模块     ## 远程主机crontab配置
mount模块     ## 挂载文件系统
find模块     ## 帮助在被管理主机中查找符合条件的文件，就像 find 命令一样
selinux模块     ## 远程管理受控节点的 selinux 的模块
```

##  四、Ansible 自动化运维操作记录

#### <font color=#0000FF> 1) 实验环境准备 </font>

|    IP 地址    |    主机名     |   角色    |  系统版本  |
| :-----------: | :-----------: | :-------: | :--------: |
| 192.168.1.100 | ansile-server | 主控节点  | centos 7.4 |
| 192.168.1.101 | ansile-agent1 | 受控节点1 | centos 7.4 |
| 192.168.1.102 | ansile-agent2 | 受控节点2 | centos 7.4 |

- 三个节点各自设置主机名

```bash
[root@ansible-server ~]# hostnamectl set-hostname ansible-server
[root@ansible-node01 ~]# hostnamectl set-hostname ansible-node01
[root@ansible-node02 ~]# hostnamectl set-hostname ansible-node02
```

- 设置主控节点与受控节点的`ssh`无密码任信任关系（`ansible`应用环境下，主控节点必须要设置`ssh`无密码跳转到受控节点的信任关系）
- 添加主控节点到受控节点的认证，首先主控节点必须要生成公私密钥对，否则不能进行免密信任关系
```bash
[root@ansible-server ~]# ssh-keygen -t rsa      ##生成密钥对，一直回车即可
[root@ansible-server ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.101    ##回车，输入远程主机密码
[root@ansible-server ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.102
```

如果受控节点数量比较多的话，可以是用`expect`进行远程`ssh`连接的免密信任关系。

####  2) Ansible 安装部署

`ansible`有两种安装方式：`yum` 和 `pip` 安装方式

`Ansible` 在 `epel` 的 `yum` 源中有提供，所以配置好 `epel` 源，直接使用 `yum` 命令安装即可。

`CentOS`系统，可以使用默认的 `yum` 源直接安装`ansible`，前提是系统能访问公网

######  yum 安装 

```bash
[root@localhost ~]# yum install -y ansible
```

###### pip 安装 

```bash
[root@localhost ~]# yum instll -y python-pip python-devel
[root@localhost ~]# pip install ansible
```

######  ansible 主要目录

```bash
配置文件目录: /etc/ansible/
执行文件目录: /usr/bin/
lib库依赖目录: /usr/lib/pythonX.X/site-packages/ansible/
Help文档目录: /usr/share/doc/ansible-X.X.X/
Man文档目录: /usr/share/man/man1/
```

###### ansible 程序文件

```bash
/usr/bin/ansible     ##命令行工具
ansible命令通用格式     ansible <host-pattern> [option] [-m module_name] [-a args]
/usr/bin/ansible-doc     ##帮助文档
/usr/bin/ansible-playbook     ##剧本执行工具
/etc/ansible/ansible.cfg     ##主配置文件
/etc/ansible/hosts     ##管理的主机清单
/etc/ansible/roles     ##角色存放处
```

###### 查看 ansible 命令帮助

```bash
[root@localhost ~]# ansible -h
```

###### 查看 ansible 支持的模块

```bash
[root@localhost ~]# ansible-doc -l        ##查看都支持哪些模块
[root@localhost ~]# ansible-doc -l | wc -l      ##查看支持多少个模块
[root@localhost ~]# ansible-doc -l | grep copy     ##查看copy模块
```

###### 查看 ansible 版本

```bash
[root@localhost ~]# ansible --version
ansible 2.4.2.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Aug  4 2017, 00:39:18) [GCC 4.8.5 20150623 (Red Hat 4.8.5-16)]
```

####  3) Ansible清单管理

- `inventory`文件通常用于定义要管理主机的认证信息，例如 `ssh` 登录用户名、密码以及 `key` 相关信息。默认 `inventory` 文件为 `/etc/ansible/host`，`ansible`模块操作命令中可以省去默认 `inventory`(即可以省去 "`-i /etc/ansible/hosts`")，如果 `inventory` 文件定义了其他路径，则在 `ansible` 模块操作命令中不能省去，要加上 "`-i inventory`" 路径。

- 如何配置 Inventory 文件？

  1. 主控节点:
  
      支持主机名通配以及正则表达式，例如: `web[1:3].test.com`
  
     支持基于非标准的 ssh 端口，例如: `web1.test.com:6666`
  
     支持定义变量，可对个别主机的特殊配置，如登录用户名、密码等

  2. 受控节点:
  
      支持嵌套组，例如: `[game:children]` 那么在 `game` 模块下面的组都会被 `game` 所包含
  
      支持指定变量，例如: `[game:vars]` 在下面指定变量

  3. `ansible` 的 `inventory` 清单文件 (`/etc/ansible/hosts`) 中配置说明:
  
      `[]` 中的名字代表组名
  
      主机 (`hosts`) 部分可以使用域名、主机名、`IP`地址表示，一般此类配置中多使用 `IP` 地址
  
      组名下的主机地址就是 `ansible` 可以管理的地址

- 下面是 `/etc/ansible/hosts`文件中清单的配置示例：
```bash
[root@ansible-server ~]# cp /etc/ansible/hosts /etc/ansible/hosts.bak
[root@ansible-server ~]# vim /etc/ansible/hosts
#可以使用受控节点的ip, 下面表示test-hosts组包括192.168.1.101 192.168.1.102 (如果ssh端口不是22, 比如是22222, 可以配置为"192.168.1.101:22222")
[test-hosts]
192.168.1.10[1:2]
  
#也可以使用受控节点的主机名, 前提是主控节点要能跟受控节点的主机名正常通信
#ansible-node1,前提是要能ping通这个主机名, 即做好/etc/hosts主机名映射
[web-nodes]
ansible-node[1:2]
```

- 测试设置是否生效

  查看 `ansible`清单里所有节点的 `uptime`情况

```bash
[root@ansible-server ansible]# ansible all -m command -a "uptime"
192.168.1.102 | SUCCESS | rc=0 >>
 17:47:09 up 2 days, 18:15,  3 users,  load average: 0.00, 0.01, 0.05

ansible-node2 | SUCCESS | rc=0 >>
 17:47:10 up 2 days, 18:15,  2 users,  load average: 0.00, 0.01, 0.05

ansible-node1 | SUCCESS | rc=0 >>
 09:49:52 up 2 days, 18:23,  3 users,  load average: 0.00, 0.01, 0.05

192.168.1.101 | SUCCESS | rc=0 >>
 09:49:52 up 2 days, 18:23,  3 users,  load average: 0.00, 0.01, 0.05
```

- 查看 `ansible`清单里 `test-hosts`组的所有节点主机名

```bash
[root@ansible-server ansible]# ansible test-hosts -m command -a "hostname"
192.168.1.102 | SUCCESS | rc=0 >>
ansible-node2

192.168.1.101 | SUCCESS | rc=0 >>
ansible-node1
```

- 查看 `ansible`清单里 `web-nodes` 组的所有节点的系统时间

```bash
[root@ansible-server ansible]# ansible web-nodes -m command -a "date"
ansible-node1 | SUCCESS | rc=0 >>
2019年 1月 02日 星期一 09:52:14 CST

ansible-node2 | SUCCESS | rc=0 >>
2019年 1月 04日 星期三 17:49:32 CST
```

- 查看`hosts`里包含了哪些主机

```bash
[root@ansible-server ansible]# ansible all --list-host
  hosts (4):
    ansible-node1
    ansible-node2
    192.168.1.101
    192.168.1.102
```

- 查看 `inventory`清单里主机列表的操作系统版本

```bash
[root@ansible-server ansible]# ansible -i /etc/ansible/hosts all -m command -a "cat /etc/redhat-release"
192.168.1.101 | SUCCESS | rc=0 >>
CentOS Linux release 7.4.1708 (Core) 

ansible-node1 | SUCCESS | rc=0 >>
CentOS Linux release 7.4.1708 (Core) 

ansible-node2 | SUCCESS | rc=0 >>
CentOS Linux release 7.4.1708 (Core) 

192.168.1.102 | SUCCESS | rc=0 >>
CentOS Linux release 7.4.1708 (Core)
```

## 五、Ansible 内置变量

| 参数                         | 用途                            | 例子                                          |
| ---------------------------- | ------------------------------- | --------------------------------------------- |
| ansible_ssh_host             | 定义 hosts ssh 地址             | ansible_ssh_host=192.168.1.1                  |
| ansible_ssh_port             | 定义 hosts ssh 端口             | ansible_ssh_port=22                           |
| ansible_ssh_user             | 定义 hosts ssh 认证用户         | ansible_ssh_user=sysadm                       |
| ansible_ssh_pass             | 定义 hosts ssh 认证密码         | ansible_ssh_pass='123456'                     |
| ansible_sudo                 | 定义 hosts sudo 用户            | ansible_sudo=root                             |
| ansible_sudo_pass            | 定义 hosts sudo 密码            | ansible_sudo_pass='123456'                    |
| ansible_sudo_exe             | 定义 hosts sudo 路径            | ansible_sudo_exe=/usr/bin/sodu                |
| ansible_connection           | 定义 hosts 连接方式             | ansible_connection=local                      |
| ansible_ssh_private_key_file | 定义 hosts 私钥                 | ansible_ssh_private_key_file=/root/key        |
| ansible_ssh_shell_type       | 定义 hosts shell 类型           | ansible_ssh_shell_type=bash                   |
| ansible_python_interpreter   | 定义 hosts 任务执行 python 路径 | ansible_python_interpreter=/usr/bin/python2.6 |
| ansible\_*_interpreter       | 定义 hosts 其他语言解析路径     | ansible\_*_interpreter=/usr/bin/ruby          |



## 六、ansible warn屏蔽命令警告

ansible监测到语法可以使用某模块，而不是shell时，会报WARNING，可以在ansible.cfg中配置command_warnings=False,或者在命令后面追加warn=false进行屏蔽

还可以在playbook中进行配置

```yml
- name: check whether unzip is installed
  ansible.builtin.shell:
    cmd: rpm -qa | grep unzip
    executable: /bin/bash
  args:
    warn: false
```

## 七、ansible failed_when使用

在正常使用了ansible剧本中的变量后，突然遇到需要在变量不符合的情况下，终端ansible的剧本，这时候就需要用到failed_when

下面脚本正常如果查询到unzip没有安装，剧本就会报错停止，通过changed_when，获取register的返回值不等于0的时候，脚本并不会报错停止，而是会继续执行

```yml
- name: check whether unzip is installed
  ansible.builtin.shell:
    cmd: rpm -qa | grep unzip
    executable: /bin/bash
  args:
    warn: false
  register: unzip_installed
  failed_when: false
  changed_when: unzip_installed.rc != 0
```

