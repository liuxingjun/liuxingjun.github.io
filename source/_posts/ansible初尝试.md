---
title: ansible初尝试
date: 2022-11-10 9:41
updated: 2023-02-01 10:30
categories: linux
---

## 安装
```bash
apt install ansible -y
```
```bash
$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/v_connliu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Jun 22 2022, 20:18:18) [GCC 9.4.0]
```
## 添加节点

### 分组节点
```ini /etc/ansible/hosts
[shudao_online]
192.168.1.10
192.168.1.11
```

### 嵌套分组
```ini /etc/ansible/hosts
[shudao:children]
shudao_online
shudao_proxy

[shudao_online]
192.168.1.10
192.168.1.11

[shudao_proxy]
192.168.2.10
192.168.2.11
```
### 使用密码登录
```ini /etc/ansible/hosts
[shudao:vars]
ansible_ssh_user='root'
ansible_ssh_pass='root'
```
使用密码登录还需要安装 `sshpass`

```bash
apt install sshpass
```

单独操作子分组时, 例如`ansible shudao_onlie  -m shell  -a 'hostnsme'` 还需要增加
```ini /etc/ansible/hosts
[shudao_onlie:vars]
ansible_ssh_user='root'
ansible_ssh_pass='root'
```

更新
操作子分组时 使用`--limit`参数,缩写`-l`即可不在子分组写相同的环境变量
例如 `ansible shudao -limit shudao_onlie -m shell  -a 'hostnsme'` 

## 操作节点

### 使用command 模块来执行查看ip命令
```bash
ansible shudao -a "ip -4 addr show eth0"
```
### 使用shell 模块来清空history
```
ansible shudao -m shell -a 'cat /dev/null > ~/.bash_history'
```
### proxy

#### 增加proxy
```bash
cmd="echo 'export http_proxy=http://10.149.9.97:3128\nexport https_proxy=http://10.149.9.97:3128' >> /etc/profile"
ansible shudao_proxy -m shell -a "$cmd"
```
#### 删除proxy
```bash
ansible shudao_proxy -a "sed -i -e '/proxy/d' /etc/profile"
```
#### apt 增加proxy
```bash
ansible shudao_proxy -m shell -a "echo 'Acquire::http::proxy \"http://10.149.9.97:3128\";\nAcquire::https::proxy \"http://10.149.9.97:3128\";' >> /etc/apt/apt.conf.d/01proxy"
```
### 修改镜像源

```bash
ansible shudao_proxy -a 'sed -i "s@/cn.archive.ubuntu.com/@/mirrors.aliyun.com/@g" /etc/apt/sources.list'
```
#### 执行aptu pdate
```bash
ansible shudao_proxy  -m apt  -a "update_cache=yes"
```

### 使用sudo

#### 全局开启
```ini /etc/ansible/ansible.cfg
[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=True
```
```bash
ansible shudao_proxy -m shell -a "sudo mkdir /data/prometheus"
```

#### 临时开启

```
ansible --help
特权升级选项:
  -K, --ask-become-pass 开启提示输入密码
  -b, --become          使用become 提升权限
```
```shell
ansible shudao_proxy -m shell -a "sudo mkdir /data/prometheus" -b -K
```