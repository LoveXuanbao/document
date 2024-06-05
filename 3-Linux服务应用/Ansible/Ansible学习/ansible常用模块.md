# 一、模块介绍

常用模块帮助文档参考：

https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

## 1.1、查看模块的使用详情

使用 `ansible-doc` 命令，`ansible-doc` 是 `ansible` 模块的文档说明，针对每个模块都有详细的用法说明及应用案例介绍，功能和 Linux 系统的 man 命令类似。该命令的使用方式如下：

- 命令格式：

```bash
ansible-doc [options] [module...]
[options]
-l --list  # 列出可用模块
-s --snippet    # 显示指定模块的 playbook 用法

# 列出支持的模块
ansible-doc -l

# 模块功能说明
ansible-doc ping
```

# 二、系统模块

## 2.1、ping 模块

### 作用

- 检查指定节点机器可用python，是否还能连通，用法很简单，不涉及参数，主机如果在线，则回复`pong`，测试连通性模块。

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m ping
192.168.1.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

[root@ansible-server ~]# ansible-doc -v ping     //可以获得该模块的说明
```

#### playbook模式

```yaml
- name: ping
  ansible.builtin.ping:
```

## 2.2、setup模块

## 2.3、selinux模块

### 作用

- 更改和配置SELinux的策略和状态

### 参数

| 参数名     | 默认值              | 参数说明                                                     |
| ---------- | ------------------- | ------------------------------------------------------------ |
| configfile | /etc/selinux/config | SELinux配置文件的路径                                        |
| policy     |                     |                                                              |
| state      |                     | SELinux的模式，disabled（关闭）、enforcing（启用）、permissive（允许） |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m selinux -a "state=disabled"
[WARNING]: SELinux state change will take effect next reboot
192.168.1.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "configfile": "/etc/selinux/config",
    "msg": "",
    "policy": "targeted",
    "reboot_required": true,
    "state": "disabled"
}
```

#### playbook

```yaml
- name: Disabled SELinux
  ansible.builtin.selinux:
    state: disabled
```

## 2.4、reboot模块

### 作用

- 重新启动服务器，等待其关闭，然后重新启动并响应命令

### 参数

| 参数名            | 默认值 | 参数说明                                                     |
| ----------------- | ------ | ------------------------------------------------------------ |
| connect_timeout   |        | 等待成功连接到受管主机后重试的最长时间                       |
| msg               |        | 重启前向用户显示的消息                                       |
| post_reboot_delay | 0      | 重新启动命令成功后等待的秒数，然后尝试验证系统是否成功重新启动 |
| pre_reboot_delay  | 0      | 重新启动前等待的秒数，作为参数传递给重新启动命令             |
| reboot_timeout    | 600    | 等待服务器冲洗能启动并响应测试命令的最长时间，此设置是针对重新启动验证和测试命令成功单独评估的，因为模块的最大执行之间为此数值的两倍 |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m reboot -a "reboot_timeout=360"
192.168.1.10 | CHANGED => {
    "changed": true,
    "elapsed": 287,
    "rebooted": true
}
```

#### playbook模式

```yaml
- name: reboot hosts
  ansible.builtin.reboot:
    reboot_timeout: 300
```



## 2.5、lvol模块

## 2.6、lvg模块

## 2.7、hostname模块

### 作用

- 设置系统的主机名

### 参数

| 参数名 | 默认值 | 参数说明                                       |
| ------ | ------ | ---------------------------------------------- |
| name   | null   | 必选参数，主机名称                             |
| use    |        | 用于更新主机名的策略，不定义的时候，会自动监测 |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m hostname -a "name=test"
192.168.1.10 | CHANGED => {
    "ansible_facts": {
        "ansible_domain": "",
        "ansible_fqdn": "test",
        "ansible_hostname": "test",
        "ansible_nodename": "test",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "name": "test"
}
```

#### playbook

```yml
- name: change hostname
  ansible.builtin.hostname:
    name: "{{ ansible_hostname }}"    # ansible_hostname为hosts文件中定义的ansible_hostname
```



## 2.8、mount模块

## 2.9、service模块

## 2.10、systemd模块

### 作用

- 在systemd管理的系统上，控制远程主机上的systemd服务。

### 参数

| 参数名        | 默认值 | 参数说明                                                     |
| ------------- | ------ | ------------------------------------------------------------ |
| daemon_reexec | no     | 在执行其他操作之前运行daemon_reexec命令，systemd管理器将序列化管理器状态 |
| daemon_reload | no     | 在执行其他操作之前运行daemon-reload，以确保systemd读取任何更改，设置为yes时，即使模块没有启动或者停止任何操作，也会运行daemon-reload |
| enabled       |        | 服务是否在启动时启动                                         |
| name          |        | 服务的名称                                                   |
| state         |        | started/stopped是幂等操作，除非必要，不会运行命令，restarted将始终重启服务，reloaded总是会重新加载 |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m systemd -a "name=rsyslog daemon-reload=yes enabled=yes state=restarted"
192.168.1.10 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "enabled": true,
    "name": "rsyslog",
    "state": "started",
    "status": {
       ...其他内容
    }
}
```

#### playbook

```yaml
- name: restart systemd-journald
  ansible.builtin.systemd:
    name: systemd-journald
    state: restarted
    enabled: true
    daemon_reload: true
```



## 2.11、systemctl模块

## 2.12、timezone模块

## 2.13、user模块

## 2.14、group模块

## 2.15、known_hosts模块

## 2.16、modprobe模块

### 作用

- 加载或卸载内核模块

### 参数

| 参数名 | 默认值  | 参数说明                                 |
| ------ | ------- | ---------------------------------------- |
| name   |         | 必填参数，要管理的内核模块的名称         |
| params |         | 可选参数，内核模块参数                   |
| state  | present | present加载内核模块， absent卸载内核模块 |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m modprobe -a "name=ip_vs state=present"
192.168.1.10 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "name": "ip_vs",
    "params": "",
    "state": "present"
}
```

#### playbook

```yaml
- name: Add the ip_vs module
  ansible.builtin.modprobe:
    name: ip_vs
    state: present
```

## 2.17、pam_limits模块

### 作用

- 修改Linux PAM 限制，

### 参数

| 参数名     | 默认值                    | 参数说明                                      |
| ---------- | ------------------------- | --------------------------------------------- |
| backup     | no                        | 可选参数，创建一个包含时间戳信息的备份文件。  |
| comment    |                           | 可选参数，                                    |
| dest       | /etc/security/limits.conf | 配置文件的路径                                |
| domain     |                           | 必选参数，限制的域，用户名、组、*等           |
| limit_item |                           | 必选参数，要做的限制，core、nofile、nproc等等 |
| limit_type |                           | 必选参数，限制类型，hard、soft                |
| value      |                           | 必选参数，限制的值                            |

### 例子

#### 命令模式

```bash
[root@ansible-server ~]# ansible -i hosts test -m pam_limits -a "dest=/etc/security/limits.conf domain='*' limit_type=hard limit_item=core value=1000000"
192.168.1.10 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "msg": "*\thard\tcore\t1000000\n"
}

# 查看文件最后一行是否添加成功
[root@ansible-server ~]# ansible -i hosts test -m shell -a "tail -1 /etc/security/limits.conf"
192.168.1.10 | CHANGED | rc=0 >>
*       hard    core    1000000
```

#### palybook

```yaml
- name: change hard core unlimits
  ansible.builtin.pam_limits:
    dest: /etc/security/limits.conf
    domain: "*"
    limit_type: hard
    limit_item: core
    value: unlimited

- name: change soft nproc ulimits
  ansible.builtin.pam_limits:
    dest: /etc/security/limits.conf
    domain: "*"
    limit_type: soft
    limit_item: nproc
    value: 1000000
```



## 2.18、pamd模块

# 三、命令模块

## 3.1、shell模块

### 作用：

用于在远程主机上执行命令





## 3.2、raw 模块

- 执行原始的命令，而不是通过模块子系统。在任何情况下，使用 `shell` 或者 `command` 命令模块也是合适的。给定原始的参数直接通过配置的远程 `shell` 运行。可返回标准输出、错误输出和返回代码。此模块没有变更处理程序支持。这个模块不需要远程系统上有 `Python`，就像脚本模块一样。此模块也支持 `windows` 目标。`raw`、`shell`、`command` 三个模块都能嗲用对象机器上的某条指令或者某个可执行文件。`raw` 和 `shell` 模块都很像，都支持管道； `command`模块不支持管道
- 注意下三个之间的微妙区别，如下会发现 `raw`模块执行的是系统原始命令，执行后主动关闭到被控制节点的连接。
```bash
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m raw -a "hostname"
192.168.1.101 | SUCCESS | rc=0 >>
ansible-node1
Shared connection to 192.168.1.101 closed.

root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m shell -a "hostname"
192.168.1.101 | SUCCESS | rc=0 >>
ansible-node1

[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m command -a "hostname"
192.168.1.101 | SUCCESS | rc=0 >>
ansible-node1
```

- `raw` 和 `shell`模块支持管道，`command`模块不支持管道

```bash
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m raw -a "cat /etc/passwd | grep root"
192.168.1.101 | SUCCESS | rc=0 >>
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
Shared connection to 192.168.1.101 closed.

[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m shell -a "cat /etc/passwd | grep root"
192.168.1.101 | SUCCESS | rc=0 >>
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin

[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m command -a "cat /etc/passwd | grep root"
192.168.1.101 | FAILED | rc=1 >>
...
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologincat: |: 没有那个文件或目录
cat: grep: 没有那个文件或目录
cat: root: 没有那个文件或目录non-zero return code
```

## 3.3、command模块





## 3.4、yum 模块

- 这个模块是`RedHat` 和`CentOS` 作为远端受控节点`OS`的时候，用的最多的模块，是`RedHat`和`CentOS`包管理工具的模块，使用`yum`软件包管理器管理软件包，其常用选项有:
  | 参数名            | 默认值  | 参数说明                                                     |
  | ----------------- | ------- | ------------------------------------------------------------ |
  | conf_file         | null    | 设置远程yum执行时所依赖的repo配置文件                        |
  | disable_gpg_check | null    | 是否关闭安装包签名的GPG检查，只有当状态为"present"和"latest"时有效 |
  | disablerepo       | null    | 安装的时候禁用某个库，不常用                                 |
  | download_only     | false   |                                                              |
  | state             | present | 表示是安装还是卸载的状态，其中present、installed、latest表示安装，absent、removed表示卸载删除，lastet表示最新版本 |
  | name              |         | 要进行操作的软件包的名字，默认最新的程序包，指明要安装的程序包，可以带上版本号，也可以传一个url或者一个本地的rpm包的路径 |
  | enablerpo         |         | 启用某个源                                                   |
> 温馨提示：要确保受控节点的 `python` 版本对应正确，否则执行下面命令会报错 (下面报错说明受控节点需要 `python2` 版本，而当前是 `python3`)

```bash
"msg": "The Python 2 bindings for rpm are needed for this module. If you require Python 3 support use the `dnf` Ansible module instead..
The Python 2 yum module is needed for this module. If you require Python 3 support use the `dnf` Ansible module instead."
```

- 安装 `httpd`

```bash
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=httpd state=latest"
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=httpd state=present"
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=httpd state=installed"

[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present"
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name="@Development tools" state=present"
```

- 删除 `httpd`

```bash
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=httpd state=absent"
[root@ansible-server ~]# ansible -i /etc/ansible/hosts all -m yum -a "name=httpd state=removed"
```

## 3.5、yum_repository模块

yum_repository模块可以帮助我们管理远程主机上的仓库

| 参数名      | 默认配置 | 参数说明                                                     |
| ----------- | -------- | ------------------------------------------------------------ |
| name        |          | 必须参数，用户指定要操作的唯一的仓库ID，也就是.repo配置文件中每个仓库对应的**[]**内的仓库ID |
| baserul     | 空       | 用于设置yum仓库的repodata目录所在的URL                       |
| description |          | 设置仓库的注释信息，对应.repo文件中的name=xxxx               |
| file        |          | 设置.repo文件的名称，不写的默认使用`name`的名称              |
| enabled     | yes      | 设置是否启用该仓库                                           |
| gpgcheck    | no       | 设置是否开启rpm包验证功能，默认值为no，表示不启用验证，yes表示启用验证 |
| gpgcakey    |          | 当gpgcheck参数设置为yes时，需要使用此参数指定的公钥验证rpm包 |
| state       | present  | 状态，present表示安装对应的yum源，absent表示删除对应的yum源  |

## 3.6、pip 模块

- 用于管理 `python` 库依赖项，为了使用 `pip` 模块，必须提供参数 `name` 或者 `requirements`，常用选项有：

| 参数名             | 默认值  | 参数说明                                                     |
| ------------------ | ------- | ------------------------------------------------------------ |
| chdir              |         | 必须参数，#执行 pip 命令前 cd 进入的目录                     |
| name               |         | 要安装的 python 库的名称或远程包的 URL                       |
| requirements       |         | 一个 pip requirements.txt 文件的路径，它应该是远程系统的本地文件，如果使用 chdir 选项，则可以将文件指定为相对路径 |
| version            |         | 指定的 python 库的安装版本，字符串类型，'version'            |
| extra_args         |         | 传递给pip的额外参数                                          |
| executable         |         | 显示可执行文件或可执行文件的路径点，用于为系统中安装的特定版本的 python 运行 pip，例如: pip-3.3，如果系统中安装了 python 2.7 和 3.3，并且想要为 python 3.3 安装运行 pip，它不能与 "virtualenv" 参数一起指定 (在2.1中添加)。默认情况下，它将采用适合用于 python 解释器的版本。 pip3 在 python 3上，pip2 或 pip 在python 2 上。 |
| virtualenv         |         | 要安装到的virtualenv目录的可选路径。 它不能与’executable’参数一起指定（在2.1中添加）。 如果virtualenv不存在，则将在安装软件包之前创建它。 可选的virtualenv_site_packages，virtualenv_command和virtualenv_python选项会影响virtualenv的创建 |
| virtualenv_command |         | 用于创建虚拟环境的命令或命令的路径名。例如 `pyvenv` ， `virtualenv` ， `virtualenv2` ， `~/bin/virtualenv` ， `/usr/local/bin/virtualenv` 。 |
| virtualenv_python  |         | 用于创建虚拟环境的Python可执行文件。例如 `python3.5` ， `python2.7` 。如果未指定，则使用用于运行ansible模块的Python版本。当 `virtualenv_command` 使用 `pyvenv` 或 `-m venv` 模块时，不应使用此参数。 |
| state              | present | 状态（present，absent，latest， forcereinstall），表示是安装还是卸载的状态. 其中present表示默认安装; lastest表示最新版本安装;absent表示卸载和删除; forcereinstall表示强制重新安装, "forcereinstall"选项仅适用于可ansible 2.1及更高版本. |



# 四、文件模块

## 4.1、file模块

## 4.2、copy模块

## 4.3、replace模块

replace模块可以根据我们指定的正则表达式替换文件中的字符串，文件中所有被匹配到的字符串都会被替换。

## 4.4、stat模块

## 4.5、lineinfile模块







## 4.6、template模块

## 4.7、unarchive模块

# 网络工具模块

## 1、url

- 与 HTTP 和 HTTPS Web 服务交互，并支持 Digest、Basic 和 WSSE HTTP 身份验证机制。

| 参数           | 默认值 | 参数说明                                                     |
| -------------- | ------ | ------------------------------------------------------------ |
| url            |        | 必填项，HTTP 或 HTTPS 的 URL                                 |
| client_cert    |        | 客户端证书；用于 SSL 客户端身份验证地 PEM 格式的证书文件     |
| client_key     |        | 客户端密钥；PEM 格式的文件，包含用于 SSL 客户端身份验证的私钥 |
| method         |        | HTTP 请求的方法                                              |
| return_content |        | 是否将响应正文作为字典结果中的内容返回                       |
| dest           |        | 下载文件的路径                                               |

## playbook

```yaml
- name: curl http nginx
  ansible.builtin.uri:
    url: http://10.202.43.114:30080
    return_content: false    # 不需要返回响应体内容  
    method: GRT              # 使用 GET 方法发送请求
  register: return_nginx     # 将响应结果保存到变量 return_nginx 中
```









# 五、ansible程序模块

## 5.1、debug模块

## 5.2、async_status模块

## 5.3、import_playbook模块

## 5.4、import_role模块

## 5.5、import_tasks模块

## 5.6、wait_for模块

## 5.7、wait_for_connection模块

### 作用

- 等待远程系统可达、可用，使用内部ansible传出和ping模块来保证正确的端到端功能

### 参数

| 参数名          | 默认值 | 参数说明                               |
| --------------- | ------ | -------------------------------------- |
| connect_timeout | 5      | 在关闭和重试之前等待连接发生的最大秒数 |
| delay           | 0      | 开始轮询之前等待的秒数                 |
| sleep           | 1      | 检查之间休眠的秒数                     |
| timeout         | 600    | 等待的最大秒数                         |

### 例子

#### palybook

```yaml
# 重启服务器
- name: Reboot host
  ansible.builtin.reboot:
  async: 300
  poll: 0

# 检查是否启动成功
- name: Wait for host
  ansible.builtin.wait_for_connection:
    connect_timeout: 20
    sleep: 5
    delay: 10
    timeout: 1200
```



