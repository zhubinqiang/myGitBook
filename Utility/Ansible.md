# Asible
[TOC]

## 主要组件
Asible core: 核心
Host inventory: 存储ssh
core modules:
custom modules:
playbook (yami, jinjia2): 剧本
connect plugin: 插件

## 安装
centos中 在 **epel.repo** 的仓库中
```sh
sudo yum -y install ansible
```

## inventory
1. ssh
2. inventory文件

```sh
ansible
```


## 配置文件
配置文件: /etc/ansible/ansible.cfg
Invertory: /etc/ansible/hosts

可以拷贝 /etc/ansible/ansible.cfg 到 ~/.ansible.cfg 作为当前用户使用
```
[defaults]
inventory      = ~/.ansible/hosts
...
```


/etc/ansible/hosts
```
[kontron]
s3n1c1
s3n1c2

[pxe]
192.168.101.100 ansible_connection=ssh        ansible_ssh_user=media
localhost ansible_connection=ssh ansible_ssh_user=user1 ansible_ssh_pass=123456
localhost ansible_connection=ssh ansible_ssh_user=user2 ansible_ssh_pass=123456 ansible_become_pass=123456
```

在本机上无密码访问远程机器
```sh
ssh-keygen

ssh-copy-id -i ~/.ssh/id_rsa.pub USER@HOST
```

## 模块帮助
ansible-doc -l
ansible-doc -s MODULE-NAME

## ansible 命令基础
ansible <host-pattern> [-f forks] [-m module_name] [-a args] [options]
-f forks: 启动的并发数
-m module_name: 
-a args: 模块特有参数

```sh
ansible s3n1c1 -m command -a 'date' -u media

ansible all -m command -a 'date' -u media

ansible -i hosts all -m command -a 'date' -u media
```

> `-u media` 是以media的账号

## 常见模块
### command
命令模块。 默认的，用于执行远程命令

### cron
```sh
ansible pxe -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job"'

ansible pxe -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job" state=absent' 
```

state: 
- present: 安装
- absent： 移除

### user
```sh
ansible pxe -m user -a 'name="user1"'

ansible pxe -m user -a 'name="user1" state=absent force'
```

### group
```sh
ansible pxe -m group -a 'name=mysql gid=306 system=yes'
```

### copy
```sh
ansible pxe -m copy -a 'src=/etc/fstab dest=/tmp/fstab.ansible owner=media mode=640'

ansible pxe -m copy -a 'content="Hello World\nWelcome" dest=/tmp/hello.txt'
```

> 1. dest 一定要绝对路径
> 2. content: 用指定生成的信息为源文件
> 3. src 是在本机上的文件

### file
```sh
ansible pxe -m file -a 'owner=user1 group=user1 mode=644 path=/tmp/fstab.ansible '
```

创建软连接
```sh
ansible pxe -m file -a 'path=/tmp/fstab.link src=/tmp/fstab.ansible state=link '
```

> `path=` 指定文件路径， 可以用`name` 或 `dest` 来代替

### ping
```sh
ansible all -m ping
```

### service
```sh
ansible pxe -m service -a 'enable=ture name=apache2 start=started'
```

> `enable=` 是否开机自动启动， 取值为true 或者 false
> `name=` 服务名称
> `state=` 状态， 取值有started， stopped， restarted

### shell
如果用到 管道 复杂命令是用shell 代替 command
```sh
ansible pxe -m shell -a 'echo password | passwd --stdin user1'
```

> `passwd --stdin user1` 对CentOS有效， Ubuntu无效

### script
执行本地的脚本, 要用相对路径指定本地脚本
```sh
ansible pxe -m script -a 'a.sh'
```

这里是脚本 `a.sh`
```sh
echo 'hello ansible from script' > /tmp/script.ansible
```

### yum
用yum 安装
```sh
## 安装
ansible s3n1c1 -m yum -a 'name=zsh'

## 卸载
ansible s3n1c1 -m yum -a 'name=zsh state=absent'
```

> `name=` 指定要安装的程序包， 可以带上版本号
> `state=` present, latest表示安装， absent表示卸载

### apt
```sh
## 安装
ansible -i hosts s5n1c1 -m apt -a "pkg=expect state=present" --become
```

### setup
收集远程主机的facts. 
每个远程主机被管理之前，将自己的相关信息，比如操作系统，IP地址等报告给Ansible主机

```sh
ansible pxe -m setup -a ''
```

### git
git clone
```sh
ansible s3n1c1 -m git -a "repo=bzhux@media-pxe3:~/WS/repos/OSConf dest=/tmp/myapp version=HEAD"
```

## Ansible 中使用YAML
### 变量
### Inventory
### 条件语句
### 迭代


## Playbook
### Inventroy
### Modules
### Ad Hoc Commands
### Playbooks
#### Tasks
任务，即调用模块完成的莫操作

#### Variables
变量

#### Templates
模板

#### Handlers
处理器，由某事件触发执行的操作

#### Roles
角色

基本结构 nginx.yml
```yaml
- hosts: pxe
  remote_user: media
  tasks:
   - name: create nginx group
     group: name=nginx system=yes gid=208
   - name: create nginx user
     user: name=nginx uid=208 group=nginx system=yes

- hosts: kontron
  remote_user: root
  tasks:
    -
```

- hosts: 主机 或者 主机组
- remote_user: 用户名
- tasks: name是取个名字，`group`, `user` 都是ansible的命令。


运行:
```sh
ansible-playbook nginx.yml
```


```yaml
tasks:
 - name: run this command and ignore the result
   shell: /usr/bin/command || /bin/true

task:
 - name: run this command and ignore the result 
   shell: /usr/bin/command
   ignore_errors: True
   register: command_result
- debug: msg="{{ command_result.stdout.split('\n') }}"
```

> `ansible-playbook -v` 之后格式化输出

apache.yaml
```yaml
- hosts: web
  remote_user: root
  tasks:
   - name: install httpd package
     yum: name=httpd state=latest
   - name: install configuration file for httpd
     copy: src=/root/conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
   - name: start httpd service
     service: enabled=true name=httpd state=started
```

## handlers
```yaml
- hosts: web
  remote_user: root
  tasks:
   - name: install httpd package
     yum: name=httpd state=latest
   - name: install configuration file for httpd
     copy: src=/root/conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
     notify:
      - restart httpd
   - name: start httpd service
     service: enabled=true name=httpd state=started
  handlers:
   - name: restart httpd
     service: name=httpd state=restarted
```

- handlers: 在 `install configuration file for httpd` 整个task执行完成之后 再去执行 `restart httpd`

> Handlers 是由通知者进行 notify, 如果没有被 notify,handlers 不会执行.
> 不管有多少个通知者进行了 notify,等到 play 中的所有 task 执行完成之后,handlers也**只会被执行一次**。
> **handlers 会按照声明的顺序执行**

## vars
```yaml
- hosts: web
  remote_user: root
  vars:
   - package: httpd
   - service: httpd
  tasks:
   - name: install httpd package
     yum: name={{ package }} state=latest
   - name: install configuration file for httpd
     copy: src=/root/conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
     notify:
      - restart httpd
   - name: start httpd service
     service: enabled=true name={{ service }} state=started
  handlers:
   - name: restart httpd
     service: name=httpd state=restarted
```

- vars: 参数, 使用时用 {{参数名}}

## when
```yaml
- hosts: kontron
  remote_user: root
  vars:
   - username: user10
  tasks:
   - name: create {{username}} user
     user: name={{username}} 
     when: ansible_fqdn == "s5n1c1.example.com"
```

- when: 把符合条件才执行任务。条件可以从 `setup` 里面挑选。

`ansible -i hosts test -m setup`

```js
"ansible_form_factor": "Other",
"ansible_fqdn": "s5n8c2.example.com",
"ansible_hostname": "s5n8c2",
"ansible_kernel": "3.10.0-693.11.6.el7.x86_64",
"ansible_lsb": {
    "codename": "Core",
    "description": "CentOS Linux release 7.4.1708 (Core)",
    "id": "CentOS",
    "major_release": "7",
    "release": "7.4.1708"
},
```

## 迭代
```yaml
- hosts: kontron
  remote_user: root
  tasks:
   - name: create several users
     user: name={{item.name}} state=present groups={{ item.groups}}
     when: ansible_fqdn == "s5n1c1.example.com"
     with_items:
      - {name: 'testuser1', groups: 'wheel'}
      - {name: 'testuser2', groups: 'root'}
```

> `item` 是固定不变的， name 是可以变的
> with_items 中的列表也可以是字典, 但引用时要使用 item.KEY

```yaml
- {name: apache, conf:confiles/httpd.conf}
- {name: php, conf:confiles/php.ini}
- {name: mysql-server, conf:confiles/my.cnf}
```

## 模板

```yaml
- hosts: web
  remote_user: root
  tasks:
   - name: copy file
     template: src=/root/ansible/template/my_temp dest=/root/my_temp.conf
```

my_temp
```
HOST = {{ ansible_fqdn }}
port = {{http_port}}
```

hosts
```
[web]
10.67.117.78 http_port=80
s5n2c2 http_port=90
s5n8c2 http_port=8080
```

### tags

```yaml
- hosts: web
  remote_user: root
  tasks:
   - name: ping
     ping:
   - name: copy file
     template: src=/root/ansible/template/my_temp dest=/root/my_temp.conf
     tags:
      - conf
```

> 为某个任务定义一个标签 `ansible-playbook --tags="conf"` 来执行playbook
> 特殊tags:always






