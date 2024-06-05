# 一、Playbook基础知识

## 1.1 playbook介绍

![image-20230825155525744](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825155525744.png)

- playbook 剧本是由一个或多个"play"组成的列表
- play的主要功能在于将预定义的一组主机，装扮成事先通过ansible中的task定义好的角色。Task 实际是调用ansible的一个module，将多个play组织在一个playbook中，即可以让它们联合起来，按事先编排的机制执行预定义的动作
- Playbook 文件是采用YAML语言编写的

ansible-playbook重要参数

1. -i：指定主机清单文件
2. -e：指定变量文件
3. ANSIBLE_CONFIG：环境变量，用于指定自己的ansible配置文件

```bash
#命令执行示例
ansible-playbook ../playbooks/9.nfs.yml -i ../config/hosts -e "@../config/config.yml"
```

## 1.2 ansible配置文件

```bash
[19:28:32 root@nexus ansible-k8s]#cat config/ansible.cfg 
[defaults]
inventory = /etc/ansible/hosts  #主机清单文件
forks          = 6     #并发数
sudo_user      = root  #sudo提权用户
remote_port    = 22    #ssh连接默认端口
roles_path = ../roles  #roles目录位置指定
#提高效率启动主机信息采集缓存,勿动
gathering = smart
fact_caching_timeout = 600 #缓存超时时间600秒
fact_caching = jsonfile    #缓存文件格式文件json
fact_caching_connection=/tmp #缓存文件位置
#提权设置，这里启用后需要在role的tasks或者playbook中指定become: yes使用
[privilege_escalation]
become=True        
become_method=sudo
become_user=root
become_ask_pass=False
```

这里面的提权设置需注意，需要配置sudo为免密

```bash
[19:38:02 root@node-1 ~]#cat /etc/sudoers
ansible ALL=(ALL)   NOPASSWD:ALL  #授权那个用户拥有sudo的免密
```

## 1.3 主机清单

inventory文件遵循INI文件风格，中括号中的字符为组名。可以将同一个主机同时归并到多个不同的组中此外，当如若目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明如果主机名称遵循相似的命名模式，还可以使用列表的方式标识各主机

内置主机变量：

- ansible_connection：远程连接方式linux一般都为ssh
- ansible_port：连接的端口ssh一般为22，如果没有变动可以不配置
- ansible_user：远程连接用户
- ansible_password：远程连接用户用户密码

```bash
[etcd]  #主机组名称
192.168.10.11:22   #主机名称后面的端口为ssh端口如果不使用默认的22端口，可自定义配置

[kube_master]
192.168.10.11

[kube_node]
192.168.10.12 ansible_connection=ssh ansible_port=22 ansible_user=wang ansible_password=123456 
192.168.10.13 ansible_connection=ssh ansible_user=root ansible_password=123456
192.168.10.14 ansible_connection=ssh ansible_user=root ansible_password=123456
192.168.10.15 ansible_connection=ssh ansible_user=root ansible_password=123456
```

**如何调用主机清单**

```bash
#1.使用-i参数直接调用即可
ansible-playbook ../playbooks/9.nfs.yml -i ../config/hosts 
#2.在配置文件中指定
[defaults]
inventory = /etc/ansible/hosts
```

## 1.4 变量使用

### 1.变量优先级

1. extra vars (在命令行中使用 -e)优先级最高
2. 然后是在inventory中定义的连接变量(比如ansible_ssh_user)
3. 接着是大多数的其它变量(命令行转换,play中的变量,included的变量,role中的变量等)
4. 然后是在主机清单中定义的其它变量
5. 然后是由系统发现的facts
6. 然后是 "role默认变量", 这个是最默认的值,很容易丧失优先权

### 2.变量文件

**如何定义**

```bash
[15:10:54 root@nexus ansible-k8s]#cat config/config.yml 
RUNTIME: "containerd"  #单个变量

ingress_host:  #变量列表，可以用来做循环遍历
- 192.168.10.13
```

**如何使用**

```bash
#1.playbook命令中直接使用-e调用变量文件
ansible-playbook ../playbooks/9.nfs.yml  -e "@../config/config.yml"

#2.在playbook文件中调用变量文件
- hosts: all
  vars_files:
    - /vars/external_vars.yml
```

### 3.主机清单定义变量

主机清单中可以定义俩种变量一个是主机变量，一个是组变量

**主机变量**：定义在主机后面，只对这个主机生效

**组变量**：

- 定义格式：`[主机组名称:vars]`
- 说明：如果主机组名称写all则表示下面定义的变量对所有主机生效，如果写test则表示面下定义的变量只对test组中的主机生效

```bash
#主机清单中定义变量
[test1]  
192.168.10.11  test=test1  #主机变量
[test2]
192.168.10.12  test=test2
[test3]
192.168.10.13  test=test2

[all:vars]  #组变量
test1="192.168.10.11"
test2="192.168.10.11"
test3="192.168.10.11"
[test1:vars] 
test1="192.168.10.11"
```

### 4.在ploybook中定义变量

```bash
- hosts: webservers
  vars:
    http_port: 80
    {{hostvars[{{groups['master'][0]}}]['hass_pass']}}
```

### 5.在role中定义默认变量

```bash
#1.直接在使用role时定义变量
- hosts: all
  roles:
    - { role: test, test_name: zhangzhuo }
#2.在role的vars或者defaults文件夹中定义变量
[15:44:39 root@nexus kube-master]#cat vars/main.yml 
test_name: "zhangzhuo"
[15:45:26 root@nexus etcd]#cat defaults/main.yml 
# etcd 集群间通信的IP和端口, 根据etcd组成员自动生成
test_name: "zhangzhuo"
```

### 6.注册变量register

register使用在task中把命令返回的结果注册到指定的变量中。

```bash
- hosts: all
  become: yes
  tasks:
    - name: 获取运行用户
      shell: whoami
      register: RUN_USER
    - name: 输出执行用户
      debug:
        msg: "{{ RUN_USER.stdout }}"
#注意如果是执行命令获取的变量他是一个列表，返回全部的结果如下，如果引用命令输出的结果需要在变量后面添加.stdout
{
        "changed": true, 
        "cmd": "whoami", 
        "delta": "0:00:00.123415", 
        "end": "2022-05-03 15:54:07.690937", 
        "failed": false, 
        "rc": 0, 
        "start": "2022-05-03 15:54:07.567522", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "root", 
        "stdout_lines": [
            "root"
        ]
}
cmd: 执行的命令
stdout：命令输出的结果
stdout_lines：命令输出的列表
```

### 7.使用Facts获取远程主机的源数据信息变量

默认情况下ansible会在运行时自己获取，无需手动调用。

```bash
#获取的变量较多这里介绍一些常用的
"ansible_architecture": "x86_64"   #系统架构
"ansible_hostname": "node-1"   #主机名称
"ansible_distribution": "CentOS"  #系统类型
"ansible_distribution_version": "7.8" #系统版本详细信息
"ansible_distribution_major_version": "7" #系统具体大版本
```

### 8.如何使用变量

```bash
#调用变量方法
{{ 变量名称 }}
#复杂变量调用方法
{{ ansible_eth0.ipv4.address }}
#调用数组的第一个元素
{{ foo[0] }}
#变量中嵌套变量使用,如果嵌套变量则无需额外写{{}}如嵌套node_name变量
错误的 {{ hostvars[{{node_name}}].if_name }}
正确的 {{ hostvars[node_name].if_name }}
```

### 9.获取其他主机注册的变量

可以用于跨playbook-role获取变量

```yaml
#获取192.168.10.11主机的if_name变量
- name: 注册变量
  set_fact: 
    net_name: "{{ hostvars['192.168.10.11'].if_name }}" 
```

# 二、playbook-role

角色是ansible自1.2版本引入的新特性，用于层次性、结构化地组织playbook。roles能够根据层次型结构自动装载变量文件、tasks以及handlers等。要使用roles只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中

运维复杂的场景：建议使用 roles，代码复用度高 

roles：多个角色的集合目录， 可以将多个的role，分别放至roles目录下的独立子目录中

roles目录结构如下所示：

![image-20230825155551257](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825155551257.png)

每个角色，以特定的层级目录结构进行组织

**Roles各目录作用**

roles/project/ :项目名称,有以下子目录

- files/ ：存放由copy或script模块等调用的文件
- templates/：template模块查找所需要模板文件的目录
- tasks/：定义task,role的基本元素，至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含
- handlers/：至少应该包含一个名为main.yml的文件；此目录下的其它的文件需要在此文件中通过include进行包含
- vars/：定义变量，至少应该包含一个名为main.yml的文件；此目录下的其它的变量文件需要在此文件中通过include进行包含
- meta/：定义当前角色的特殊设定及其依赖关系,至少应该包含一个名为main.yml的文件，其它文 件需在此文件中通过include进行包含
- default/：设定默认变量时使用此目录中的main.yml文件，比vars的优先级低。

```bash
#role目录结构
roles/     #role的归档目录
   mysql/  #具体的一个role，可以理解为role的名称
     files/  #role的静态文件位置，一般存放静态文件或者压缩包等
     templates/ #j2的模板文件位置
     tasks/     #task任务文件位置，必须存在一个main.yml主文件
     handlers/
     vars/      #变量文件存放位置,必须存在一个main.yml
     defaults/  #默认变量文件存放位置,必须存在一个main.yml
     meta/      #角色依赖，必须存在一个main.yml
```

说明：

- 一个role中必须存在的文件夹为task，其他的文件如果不使用可以不进行创建
- 所有copy tasks可以引用roles/x/files/中的文件，不需要指明文件的路径
- 所有script tasks可以引用 roles/x/files/ 中的脚本，不需要指明文件的路径
- 所有template tasks可以引用 roles/x/templates/ 中的文件，不需要指明文件的路径。
- 所有include tasks可以引用 roles/x/tasks/ 中的文件，不需要指明文件的路径。
- 如果 roles/x/meta/main.yml 存在, 其中列出的 “角色依赖” 将被添加到 roles 列表中
- 如果 roles/x/vars/main.yml 存在, 其中列出的 variables 将被添加到 play 中
- 如果 roles/x/tasks/main.yml 存在, 其中列出的 tasks 将被添加到 play 中

## 2.1 playbook调用角色

示例如下：

```bash
#方案1
- hosts: all         #执行此role的host
  roles:             #调用的role名称
    - mysql
    - nginx
#方案2
- hosts: all
  roles:
    - { role: nginx, username: nginx }  #调用role时给role传递变量
#方案3
- hosts: all
  remote_user: root
  roles:
    - { role: nginx, username: nginx, when: ansible_distribution_major_version == '7' }  #根据when决定是否执行role
```

## 2.2 playbook中直接定义task

playbook中也可以直接定义task，如果定义了task则无法定义roles，如果在此定义roles则tasks会被覆盖无法执行。

```bash
- hosts: all         
  tasks:            
    - name: 测试1
      shell: echo "test1"
    - name: 测试2
      shell: echo "test2"   
```

## 2.3 管理节点过多导致的超时时间问题解决方法

如果hosts指定为all且主机数量较多，默认情况下，Ansible将尝试并行管理playbook中所有的机器。对于滚动更新用例，可以使用serial关键字定义Ansible一次应管理多少主机，还可以将serial关键字指定为百分比，表示每次并行执行的主机数占总数的比例

```bash
- hosts: all
  serial: 1    #每次只同时处理2个主机,将所有task执行完成后,再选下2个主机再执行所有task,直至所有主机处理完成
  serial: "20%"  #每次只同时处理20%的主机
  roles:
  - test
```

# 三、变量使用

## 3.1 变量优先级

1. extra vars (在命令行中使用 -e)优先级最高
2. 然后是在inventory中定义的连接变量(比如ansible_ssh_user)
3. 接着是大多数的其它变量(命令行转换,play中的变量,included的变量,role中的变量等)
4. 然后是在主机清单中定义的其它变量
5. 然后是由系统发现的facts
6. 然后是 "role默认变量", 这个是最默认的值,很容易丧失优先权

## 3.2 变量文件

**如何定义**

```bash
[15:10:54 root@nexus ansible-k8s]#cat config/config.yml 
RUNTIME: "containerd"  #单个变量

ingress_host:  #变量列表，可以用来做循环遍历
- 192.168.10.13
```

**如何使用**

```bash
#1.playbook命令中直接使用-e调用变量文件
ansible-playbook ../playbooks/9.nfs.yml  -e "@../config/config.yml"

#2.在playbook文件中调用变量文件
- hosts: all
  vars_files:
    - /vars/external_vars.yml
```

## 3.3 主机清单定义变量

主机清单中可以定义俩种变量一个是主机变量，一个是组变量

**主机变量**：定义在主机后面，只对这个主机生效

**组变量**：

- 定义格式：`[主机组名称:vars]`
- 说明：如果主机组名称写all则表示下面定义的变量对所有主机生效，如果写test则表示面下定义的变量只对test组中的主机生效

```bash
#主机清单中定义变量
[test1]  
192.168.10.11  test=test1  #主机变量
[test2]
192.168.10.12  test=test2
[test3]
192.168.10.13  test=test2

[all:vars]  #组变量
test1="192.168.10.11"
test2="192.168.10.11"
test3="192.168.10.11"
[test1:vars] 
test1="192.168.10.11"
```

## 3.4 在ploybook中定义变量

```bash
- hosts: webservers
  vars:
    http_port: 80
```

## 3.5 在role中定义默认变量

```bash
#1.直接在使用role时定义变量
- hosts: all
  roles:
    - { role: test, test_name: zhangzhuo }
#2.在role的vars或者defaults文件夹中定义变量
[15:44:39 root@nexus kube-master]#cat vars/main.yml 
test_name: "zhangzhuo"
[15:45:26 root@nexus etcd]#cat defaults/main.yml 
# etcd 集群间通信的IP和端口, 根据etcd组成员自动生成
test_name: "zhangzhuo"
```

## 3.6 注册变量register

register使用在task中把命令返回的结果注册到指定的变量中。

```bash
- hosts: all
  become: yes
  tasks:
    - name: 获取运行用户
      shell: whoami
      register: RUN_USER
    - name: 输出执行用户
      debug:
        msg: "{{ RUN_USER.stdout }}"
#注意如果是执行命令获取的变量他是一个列表，返回全部的结果如下，如果引用命令输出的结果需要在变量后面添加.stdout
{
        "changed": true, 
        "cmd": "whoami", 
        "delta": "0:00:00.123415", 
        "end": "2022-05-03 15:54:07.690937", 
        "failed": false, 
        "rc": 0, 
        "start": "2022-05-03 15:54:07.567522", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "root", 
        "stdout_lines": [
            "root"
        ]
}
cmd: 执行的命令
stdout：命令输出的结果
stdout_lines：命令输出的列表
```

## 3.7 使用Facts获取远程主机的源数据信息变量

默认情况下ansible会在运行时自己获取，无需手动调用。

```bash
#获取的变量较多这里介绍一些常用的
"ansible_architecture": "x86_64"   #系统架构
"ansible_hostname": "node-1"   #主机名称
"ansible_distribution": "CentOS"  #系统类型
"ansible_distribution_version": "7.8" #系统版本详细信息
"ansible_distribution_major_version": "7" #系统具体大版本
```

## 3.8 如何使用变量

```bash
#调用变量方法
{{ 变量名称 }}
#复杂变量调用方法
{{ ansible_eth0.ipv4.address }}
#调用数组的第一个元素
{{ foo[0] }}
```

# 四、role中的tasks

role中的tasks文件夹下必须有一个main.yml，在playbook-role调用role时一开始运行的就是main.yml，其他的task需要进行调用才可以被执行。

**main.yml示例**

```bash
[16:35:01 root@nexus tasks]#cat main.yml
- name: 创建证书存放目录   #任务名称
  file: name={{ item }} state=directory  #这里为调用的模块配置
  
- name: 安装cfssl证书工具  #可以依次定义多个，会按照定义顺序执行
  unarchive: copy=yes src=cfssl_1.6.0.tar.gz dest=/usr/b

- import_tasks: etcd.yml  #调用其他task
```

## 4.1 task中的循环

通常你想在一个任务中干很多事,比如创建一群用户,安装很多包,或者重复一个轮询步骤直到收到某个特定结果，可以使用循环在一个任务中执行多个相同的操作。

### 1.标准循环with_items

```bash
#循环一个值
- hosts: all 
  become: yes 
  tasks:
    - name: 创建用户
      user: name={{ item }}
      with_items:
      - name1
      - name2 
#循环数组变量，与上面示例效果一致
- hosts: all
  become: yes
  vars:
    user_name:
    - name1
    - name1
  tasks:
    - name: 创建用户
      user: name={{ item }}
      with_items: "{{ user_name }}"
#循环多个值
#测试连接
- hosts: all
  become: yes
  tasks:
    - name: 创建用户
      user: name={{ item.name }} state=present groups={{ item.groups }}
      with_items:
      - { name: 'testuser1', groups: 'wheel' }
      - { name: 'testuser1', groups: 'wheel' }
```

### 2.嵌套循环with_nested

```bash
#创建目录并且修改权限
- hosts: all
  become: yes
  tasks:
    - name: 创建文件
      file: path=/opt/{{ item[0] }} owner={{ item[1] }}  state=directory
      with_nested: 
        - [ 'test-1', 'test-2' ]
        - [ 'name1', 'name2', ]
#最后文件的所属用户为name2，在test-1与test-2目录上面都会循环2次修改权限的操作
```

### 3.Do-Until循环

比如有时你会判断上一个任务是否执行成功，比如判断一个服务是否启动成功

- delay：延迟多少秒
- retries：重试多少次
- until：判断条件

```bash
- name: 轮询等待kube-apiserver启动                         
  shell: "systemctl is-active kube-apiserver.service"
  register: api_status
  until: api_status.stdout == "active"
  retries: 3
  delay: 10

- name: 轮询等待kube-node节点注册成功
  shell: "kubectl get node | grep -o {{ hostvars[item]['inventory_hostname'] }}"
  register: kubelet_status
  with_items: 
  - "{{ groups['kube_node'] }}"
  - "{{ groups['kube_master'] }}"
  until: "kubelet_status.stdout in K8S_NODES"
  retries: 4
  delay: 2
```

## 4.2 条件选择when

可以用于roles与tasks或者include，在执行任务时进行条件判断，以确定是否运行roles或者某一个tasks。

**条件说明**

- ==：等于
- !=：不等于
- is not defined：判断变量没有定义
- is defined：判断变量已经定义
- and：多个条件判断

```bash
#roles中配置
- hosts: all
 remote_user: root
 roles:
   - { role: nginx, when: ansible_distribution_major_version == '7' }
#tasks中配置
- name: 测试
  shell: "echo 1"
  when: ansible_distribution_major_version == '7'
#include中配置
- include: tasks/sometasks.yml
  when: ansible_os_family == "RedHat" 
  
#判断存在不存在
tasks:
  - shell: echo "foo 存在"
    when: foo is defined
  - shell: echo "foo 不存在"
    when: foo is not defined

#判断布尔型变量
vars:
  epic: true | false
  
tasks:
  - shell: echo "真"
    when: epic
tasks:
  - shell: echo "假"
    when: not epic
```

## 4.3 忽略错误

```bash
- name: this will not be counted as a failure
  command: /bin/false
  ignore_errors: yes
```

## 4.4 become提权

become: 可用于Host和task中。执行任务时进行提权，需要在ansible中指定提权设置。

```yaml
#playbook定义
- hosts: web1:web2
  become: yes

#在其中一个task定义
- name: test connection
  ping:
  become: yes
```

## 4.5 使用connection本地执行tasks

使用connection可以在本机执行tasks，可以定义在task也可以定义在playbook中

**示例**

```bash
#在playbook中定义，如果定义在这里整个role都会在本地执行
- hosts: all
  connection: local   #写到这里表现tasks中所有的任务都在本机执行
  roles:
  - test
#在task中定义，如果定义在task的某一个任务只会在执行这个任务时在本地执行
- name: node
  shell: hostname
  connection: local   
```

## 4.6 handlers和notify

Handlers本质是task list ，类似于MySQL中的触发器触发的行为，其中的task与前述的task并没有本质上的不同，主要用于当关注的资源发生变化时，才会采取一定的操作。而Notify对应的action可用于在每个play的最后被触发，这样可避免多次有改变发生时每次都执行指定的操作，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调用handler中定义的操作

**注意**

- 如果多个task通知了相同的handlers， 此handlers仅会在所有tasks结束后运行一次 
- 只有notify对应的task发生改变了才会通知handlers， 没有改变则不会触发handlers

示例：

```bash
#task中定义
- name: 获取主机名称
  shell: hostname
  register: HOSTNAME
  notify:    
    - echo hostname  
#handlers需要定义在handlers的main.yml文件中
[18:46:14 root@nexus ping]#cat handlers/main.yml 
- name: echo hostname
  debug: 
    msg: "{{ HOSTNAME.stdout }}"
```

## 4.7 关闭 changed 状态

当确定某个task不会对被控制端做修改时,可以通过 changed_when: false 关闭changed状态

```yaml
- name: error
  command: /bin/false
  ignore_errors: yes
  changed_when: false   #关闭changed状态  
```



# 五、role中的templates

模板是一个文本文件，可以做为生成文件的模版，并且模板文件中还可嵌套jinja语法

## 5.1 jinja2语言

**官方网站**

http://jinja.pocoo.org/

https://jinja.palletsprojects.com/en/2.11.x/

jinja2 语言支持多种数据类型和操作

```bash
字面量，如: 字符串：使用单引号或双引号,数字：整数，浮点数
列表：[item1, item2, ...]
元组：(item1, item2, ...)
字典：{key1:value1, key2:value2, ...}
布尔型：true/false
算术运算：+, -, *, /, //, %, **
比较操作：==, !=, >, >=, <, <=
逻辑运算：and，or，not 
流表达式：For，If，When
```

**字面量**

表达式最简单的形式就是字面量。字面量表示诸如字符串和数值的 Python 对象。如"Hello World" 双引号或单引号中间的一切都是字符串。无论何时你需要在模板中使用一个字符串（比如函数调用、过滤器或只是包含或继承一个模板的参数），如42，42.23

数值可以为整数和浮点数。如果有小数点，则为浮点数，否则为整数。在 Python 里， 42 和 42.0 是不一样的

**算术运算**

Jinja 允许用计算值。支持下面的运算符

```bash
+：把两个对象加到一起。通常对象是素质，但是如果两者是字符串或列表，你可以用这种方式来衔接它们。无论如何这不是首选的连接字符串的方式！连接字符串见 ~ 运算符。 {{ 1 + 1 }} 等于 2 
-：用第一个数减去第二个数。 {{ 3 - 2 }} 等于 1 
/：对两个数做除法。返回值会是一个浮点数。 {{ 1 / 2 }} 等于 0.5 
//：对两个数做除法，返回整数商。 {{ 20 // 7 }} 等于 2 
%：计算整数除法的余数。 {{ 11 % 7 }} 等于 4 
*：用右边的数乘左边的操作数。 {{ 2 * 2 }} 会返回 4 。也可以用于重 复一个字符串多次。 {{ '=' * 80 }} 会打印 80 个等号的横条\
**：取左操作数的右操作数次幂。 {{ 2**3 }} 会返回 8
```

**比较操作符**

```bash
== 比较两个对象是否相等
!= 比较两个对象是否不等
> 如果左边大于右边，返回 true 
>= 如果左边大于等于右边，返回 true 
< 如果左边小于右边，返回 true 
<= 如果左边小于等于右边，返回 true 
```

**逻辑运算符**

```bash
对于 if 语句，在 for 过滤或 if 表达式中，它可以用于联合多个表达式
and 如果左操作数和右操作数同为真，返回 true 
or 如果左操作数和右操作数有一个为真，返回 true 
not 对一个表达式取反
(expr)表达式组
true / false true 永远是 true ，而 false 始终是 false 
```

## 5.2 template

template功能：可以根据和参考模块文件，动态生成相类似的配置文件 

template文件必须存放于templates目录下，且命名为 .j2 结尾 

yaml/yml 文件需和templates目录平级，目录结构如下示例：

```bash
[16:31:15 root@web1 nginx]#tree 
.
├── nginx.retry
├── nginx.yaml
└── templates
    ├── index.html.j2
    └── nginx.conf.j2
```

### 1.template模板使用

```bash
- name: teplate config to remote hosts
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

## 5.3 使用流程控制for和if

for与if不止在template中可以使用，在vars与defaults中也可以使用

### 1.for循环

**说明**

```bash
{% for i in test %} #循环定义
<h1>{{ i }}</h1>
{% else %}     #如果变量为空执行
<h1>is null</h1>
{% endfor %}  #结束循环
```

**循环示例**

```bash
#循环变量或者列表
{% for index_name in nginx_user %}
<h1>{{ index_name }}</h1>
{% endfor %}

#循环主机组列表
{% for node in groups['test'] %}
<h1>{{ node }}</h1>
{% endfor %}

#循环控制输出
{% for nide in name %}
<h1>{{ nide.hostname }}</h1>
<h1>{{ nide.root }}</h1>
<h1>{{ nide.name }}</h1>
{% endfor %}
```

**循环变量的说明**

```bash
#单个变量一次循环一个字符
nginx_user: nginx  
#列表变量一次循环一个元素
ip:  
- 192.168.10.181 
- 192.168.10.182
- 192.168.10.183
#这样定义变量可以控制输入选择
name:    
  - hostname: "{{ ansible_nodename }}"
    root: root
    name: zhangzhuo
```

### 2.if判断

在模版文件中还可以使用 if条件判断，决定是否生成相关的配置信息

**说明**

```bash
{% if test == 'true' %}   #判断成立执行
<h1>is true</h1>
{% elif test == 'false' %} #多条件，可以不写，即为一个条件
<h1>is nide</h1>
{% else %}   #所有判断条件不成立执行
<h1>is flase</h1>
{% endif %}  #结束
```

## 5.4 字符串处理函数

ansible是python实现的所以python中的一些字符串方法也是可以在templates、vars中使用的

rstrip()：对字符串右边进行删除，并返回处理后的值

lower()：把字符串字母转换为小写

upper()：把字符串字母转换为大写

文档说明：https://www.w3school.com.cn/python/python_ref_string.asp

**vars示例**

```bash
#rstrip使用，返回192.168.10
TEST: "192.168.10.71"
TEST_NEW: "{{ TEST.rstrip('.71') }}"
#lower使用，返回zhangzhuo001
TEST: "ZHANGZHUO001"
TEST_NEW: "{{ TEST.lower() }}"
```

# 六、模块介绍

## 6.1 Ansible常用模块

2015年底270多个模块，2016年达到540个，2018年01月12日有1378个模块，2018年07月15日1852 个模块,2019年05月25日（ansible 2.7.10）时2080个模块，2020年03月02日有3387个模块

虽然模块众多，但最常用的模块也就2，30个而已，针对特定业务只用10几个模块 

常用模块帮助文档参考：

https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

## 6.2 如何查看一个模块的使用详情

使用`ansible-doc`命令，可以查看模块的使用详情，具体如下

```bash
#格式
ansible-doc [options] [module...]
-l --list   #列出可用模块
-s, --snippet #显示指定模块的playbook片段


#列出所有模块
ansible-doc -l
#查看指定模块帮助用法1
ansible-doc ping
#查看指定模块帮助用法2
ansible-doc -s ping
```

## 6.3 Shell模块

功能：用shell执行命令,支持各种符号,比如:*,$, > 

注意：此模块不具有幂等性

**tasks范例**：

```yml
- name: 获取运行用户
  shell: whoami
```

注意：调用bash执行命令 类似 cat /tmp/test.md | awk -F'|' '{print \$1,$2}' &> /tmp/example.txt 这些复杂命令，即使使用shell也可能会失败，解决办法：写到脚本时，copy到远程，执行，再把需要的结果拉回执行命令的机器

## 6.4 Script模块

**功能**：在远程主机上运行ansible服务器上的脚本(无需执行权限) 

**注意**：

- 此模块不具有幂等性
- roles中需要把执行的脚本放置在files目录下

**范例：**

```bash
#在files准备脚本文件
[19:00:33 root@nexus ping]#cat files/echo.sh 
#!/bin/bash
echo "zhangzhuo"
#task使用模块
- name: 执行脚本
  script: echo.sh chdir=/opt
#参数说明
chdir  #执行脚本的目录
```

## 6.5 Copy模块

功能：从ansible服务器主控端复制文件到远程主机

roles中使用需要把文件放置到files中

```bash
#示例
- name: 拷贝文件
  copy: src=echo.sh dest=/root/1.sh owner=name1 group=name1 mode=600 backup=yes
```

**参数介绍**

```bash
src  #复制目标文件或目录位置
dest #复制到目的的位置
content #生成文件
owner  #复制到目的地的文件修改所有用户
group  #复制到目的地的文件修改所有组
mode   #复制到目的地修改文件权限
backup #文件存在是否备份
```

## 6.6 File模块

功能：设置文件属性,或者创建目录、文件、软链接等

**范例：**

```yaml
#示例
- name: 创建文件
  file: path=/root/zz  state=touch owner=name1 mode=755 
```

**参数选项**

```bash
path  #目标文件或目录位置
state #文件的类型 touch：创建普通文件 directory：目录 link：软连接  删除：absent
owner #所属用户
group #所属组
mode  #文件权限
recurse #递归操作，必须是目录
```

## 6.7 unarchive 模块

功能：解包解压缩 实现有两种用法： 

1. 将ansible主机上的压缩包传到远程主机后解压缩至特定目录，设置copy=yes 
2. 将远程主机上的某个压缩包解压缩到指定路径下，设置copy=no 

如果在role中使用压缩包也需要放置到files目录。

**常见参数**

```bash
copy #默认为yes，当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，如果设置为copy=no，会在远程主机上寻找src源文件
remote_src #和copy功能一样且互斥，yes表示在远程主机，不在ansible主机，no表示文件在ansible主机上
src #源路径，可以是ansible主机上的路径，也可以是远程主机(被管理端或者第三方主机)上的路径，如果是远程主机上的路径，则需要设置copy=no
dest #远程主机上的目标路径
mode #设置解压缩后的文件权限
owner #设置解压后文件的所属用户
group #设置解压后文件所属组
```

**范例**

```bash
#压缩包在本机，解压到远端
- name: 部署helm二进制文件
  unarchive: copy=yes src=helm-v3.7.2-linux-amd64.tar.gz dest=/usr/bin/ mode=755 
#从远程下载压缩包解压
- name: 部署helm二进制文件
  unarchive: remote_src=yes src=https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz dest=/usr/bin/ mode=755 owner=root group=root
```

## 6.8 Cron模块

功能：计划任务 

支持时间：minute，hour，day，month，weekday

**范例**

```bash
- name: 设置定时任务
  cron: hour=2 minute=30 weekday=1-5 name="log" job=hostname.sh
#删除任务
- name: 设置定时任务
  cron: name="log" state=absent
```

**参数介绍**：

```bash
minute  #设置分钟，默认*
hour    #设置小时，默认*
day     #设置天，默认*
month   #设置月，默认*
weekday #设置周，默认*
job     #指定定时任务命令或者脚本文件
name    #计划任务名称
state   #当计划任务有名称时，我们可以根据名称修改或删除对应的任务，当删除计划任务时，需要将 state 的值设置为 absent
```

## 6.9 Yum和Apt模块

功能： 

- yum 管理软件包，只支持RHEL，CentOS，fedora，不支持Ubuntu其它版本 
- apt 模块管理 Debian 相关版本的软件包

**范例**

```bash
#安装
- name: 安装软件
  yum: name=httpd,mysql state=present conf_file=/tmp/local.repo
#卸载
- name: 安装软件
  yum: name=httpd,mysql state=absent
#示例
- name: 安装 centos7 基础软件
  yum:
    name: ['curl', 'ipvsadm', 'ipset', 'iptables', 'sysstat', 'libseccomp', 'rsync', 'wget', 'psmisc', 'vim', 'net-tools', 'telnet', "chrony","bash-completion"]
    state: present
    conf_file: /tmp/local.repo
    disablerepo: "*"
    enablerepo: loacl
```

**参数介绍**

```bash
name   #安装软件名称，多个以,隔开
state  #动作，present安装，absent卸载
conf_file #指定yum仓库配置文件
disablerepo #指定关闭某些仓库，可以使用*表示所有仓库
enablerepo  #指定开启某些仓库，可以使用*表示所有仓库
```

## 6.10 Service模块

功能：管理服务,启动或关闭服务	

**范例**

```bash
- name: 启动服务
  service: name=kubelet state=restarted enabled=yes
```

**参数说明**

```
name  #服务名称
state #动作，started启动，stoped关闭，restarted重启
enabled  #设置开机自启
```

## 6.11 systemd模块

管理systemd服务，跟service基本功能一样

**范例**

```bash
- name: 重新加载所有服务配置
  systemd: daemon_reload=yes
```

**参数**

```bash
name  #服务名称，需要指定完整的服务名称如httpd.service
enabled #开启自启
daemon_reload  #重新加载systemd服务配置
state  #动作，started启动，停止stopped，重启restarted，reloaded重新加载服务配置
```

## 6.12 User模块

功能：管理用户 

**范例**

```bash
- name: 创建用户
  user: name=zhangzhuo comment="test zhangzhuo" uid=2020 home=/home/cy group=root shell=/sbin/nologin system=yes create_home=no home=/data/nginx 
```

**参数说明**

```bash
name  #用户名称
comment #用户说明
uid  #用户uid
group #用户主组
groups #用户附加组
shell  #用户默认使用那个shell
system  #是否是系统用户
create_home #是否创建家目录
home #家目录位置
remove #删除用户时，把用户相关目录也删除
```

## 6.13 Group 模块

功能：管理组 

**范例**

```bash
#创建组
- name: 创建用户
  group: name=zz  gid=2020 system=yes
#删除组
- name: 创建用户
  group: name=zz  gid=2020 system=yes state=absent
```

## 6.14 lineinfile模块

ansible在使用sed进行替换时，经常会遇到需要转义的问题，而且ansible在遇到特殊符号进行替换时， 存在问题，无法正常进行替换 。其实在ansible自身提供了两个模块：lineinfile模块和replace模块，可以方便的进行替换

一般在ansible当中去修改某个文件的单行进行替换的时候需要使用lineinfile模块

**regexp参数** ：使用正则表达式匹配对应的行，当替换文本时，如果有多行文本都能被匹配，则只有最后面被匹配到的那行文本才会被替换，当删除文本时，如果有多行文本都能被匹配，那么这些行都会被删除。

**如果想进行多行匹配进行替换需要使用replace模块**

**范例**

```bash
#替换内容，只替换匹配到的最后一行
- name: 替换内容，只替换匹配到的最后一行
  lineinfile:
    path: /root/1.sh
    regexp: "z"
    line: "zhangzhuo" #替换的是整行内容
#删除行内容，匹配到的行全部删除
- name: 删除文本内容
      lineinfile:
    path: /root/1.sh
    state: absent
    regexp: "^#" 
```

## 6.15 replace模块

该模块有点类似于sed命令，主要也是基于正则进行匹配和替换，建议使用

**范例**

```bash
- name: 替换内容，匹配到的行全部替换
  replace:
    path: /root/1.sh
    regexp: "^zhangzhuo(.*)"
    replace: zz\1
```

## 6.17 setup模块

功能： setup 模块来收集主机的系统信息，这些 facts 信息可以直接以变量的形式使用，但是如果主机较多，会影响执行速度，可以使用 gather_facts: no 来禁止 Ansible 收集 facts 信息,一般会自动执行无需配置

```bash
- name: 获取主机信息
  setup:
```

## 6.18 debug模块

此模块可以输出信息，一般用于排错

**范例**

```bash
- name: 输出变量
  debug:
    msg: "{{ RUN_USER }}"
```