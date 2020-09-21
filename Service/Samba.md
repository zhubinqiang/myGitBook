# Samba 
[TOC]

## 安装
### Ubuntu
```bash
apt-get install samba
```

### CentOS
```bash
yum install samba samba-common
```


## 配置文件
/etc/samba/smb.conf

|参数|说明|
|:---|:---|
|[global]|全局参数|
|workgroup=MYGROUP| 工作组名称|
|server string=Samba Server Version %v|服务器介绍， %v:版本号|
|log file=/var/log/samba/log.%m|log地址 %m:来访的主机名|
|max log size=50|日志最大容量50Kb|
|security=user|安全验证，总共有[4种][1]|
|passdb backend=tdbsam| 定义用户后台类型， 共[3种][2]|
|load printers=yes|SMB 启动时是否共享打印设备|
|cpus options = raw| 打印机选项|
|[homes]|共享参数|
|comment = Home Directories|描述信息|
|browseable=no|是否在"网上邻居"中可见|
|writable=yes|定义是否可写入， "read only"相反|
|path=/var/spool/samba| 共享文件的实际路径|
|guest ok=no|是否所有人可见|

security:
- share: 来访主机无需口令
- user: 需要由SMB服务验证来访主机提供的口令后才能访问
- server:使用独立的远程主机来访问主机提供的口令(集中管理账号)
- domain: 使用PDC来验证

passdb backend:
- smbpasswd:使用SMB服务的smbpasswd命令给用户设置SMB命令
- tdbsam: 创建数据库文件并使用pdbedit建立SMB独立用户
- ldapsam: 基于LDAP服务来验证

## 验证模式
### tdbsam 模式
```
    security = user
    passdb backend = tdbsam
```

```bash
mkdir /database
```

```
[database]
comment = Do not arbitrarily modify the database file
path = /database
public = no
writable = yes
```

```bash
useradd smbuser
pdbedit -a -u smbuser
```

### smbpasswd 模式
```
    security = user
    passdb backend = smbpasswd
```

```bash
smbpasswd -a smbuser2
```


## samba 服务命令
```bash
sudo systemctl enable smb.service   # 开机启动
sudo systemctl enable nmb.service
sudo systemctl reload smb.service   # 重载
```


## sample
```
[WS]
    comment =  share workspace
    path = /home/user1/WS
    browseable = yes
    guest ok = yes
    writable = yes
    valid users = user1
```


## Windowns 访问
```batch
\\IP\WS
```

## Linux 访问
```bash
sudo mount -t cifs //IP/WS /mnt -o user=[USER],passwd=[PASSWORD]

# 像FTP客户端一样使用smbclient, 不加-U 以当前用户名访问
smbclient //[IP]/WS  -U [USER]%[PASSWORD]

# 查看共享文件夹
smbclient -L [IP] -U [USER]%[PASSWORD]
```




