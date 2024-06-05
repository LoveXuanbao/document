## 批量重启服务器

```bahs
ansible -i kvm_hosts 172.16.104.18 -m shell -a "shutdown -h now"
```

## 批量修改主机名

- hosts文件中格式应为:   **主机名    ansible_host=IP地址**

```bash
ansible -i zeta_hosts all -m hostname -a "name={{ inventory_hostname }}"
```



批量添加sudo权限

```shell
# hosts文件中配置
ansible_become_pass=XXXX

ansible -i hosts all -m lineinfile -a "path=/etc/sudoers insertafter='^#\s*%wheel\s+ALL=\(ALL\)\s+NOPASSWD:\s+ALL' line='dfuser  ALL=(ALL)       NOPASSWD: ALL'"
```



## 拷贝文件到远程机解压，并执行脚本

```yml
- hosts: all
  tasks:
  - name: copy soc_deploy.zip
    unarchive:
      src: ./soc_deploy.zip
      dest: /tmp/

  - name: bash upgrade.sh
    shell:
      cmd: bash upgrade_agent.sh
      chdir: /tmp    ## 指定执行目录
```

## 查看网卡相关信息

```bash
ansible -i new_kafka_es_hosts node -m setup -a "filter=ansible_default_ipv4"
```

## 判断文件是否存在某一行内容

```
ansible -i /data/test/hosts all -m command -a "grep -Fxq '11.53.101.81 harbor.service.moebius.com' /etc/hosts" | grep FAILED
```

```
- hosts: all
  tasks:
  - name: Check file Content /etc/hosts
    command: grep -Fxq "11.53.101.81 harbor.service.moebius.com" /etc/hosts
    register: check_hosts
    check_mode: no
  #  ignore_errors: yes
    changed_when: no

  #- name: test /etc/hosts
  #  debug: msg="test"
  #  when: check_hosts.rc == 0
```

## 批量修改用户名密码

### 方法一

```shell
---
- hosts: ha
  sudo: yes
  vars:
    user: hangtyw
    password: "$1$7H1VPRkc$nosotVUh//6YBzpYtY58U0"			## 加密后的密码
    password: FrB3aZnNAHf0K@7
  remote_user: sysadm
  tasks:
  - name: Add user {{ user }}   ## 添加用户
    user: name={{user}} comment="ceph user" password={{ password }}   ## 用加密后的密码修改密码
    shell: useradd {{ user }} && echo {{ password }} | passwd --stdin {{ user }}
  - name: Config /etc/sudoers  ## 添加 sudo su 权限
    lineinfile: dest=/etc/sudoers state=present  line='{{item}}' validate='visudo -cf %s'
    with_items:
           - "{{ user}} ALL=(ALL) NOPASSWD: ALL"
           - "Defaults: {{user}}  !requiretty"

  - name: change user passwd   ## 修改密码
    user: name={{ item.user }} password={{ item.password | password_hash('sha512') }}  update_password=always
    with_items:
          - { user: 'sysadm', password: 'kRYVRDXKXY4*oLg' }
          - { user: 'root', password: '2IU$3Pe8zHuIUOh' }
```

### 方法二
```shell
---
 tasks file for change-password
- name: change root password
  user:
    name: root
    state: present
    password: "{{ root_passwd | password_hash('sha512') }}"  ##  需要定义变量：root_passwd
    update_password: always

- name: create user guest
  user:
    name: sysadm
    state: present
    password: "{{ guest_passwd | password_hash('sha512') }}"  ## ##  需要定义变量：guest_passwd
    update_password: always
```
