# CentOS7

[TOC]

## 几个yum Repo URL
```
http://li.nux.ro/repos.html

http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm

https://copr.fedorainfracloud.org/coprs/pypa/pypa/repo/epel-7/pypa-pypa-epel-7.repo

https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

## 配置yum 代理
/etc/yum.conf
```
proxy=http://proxy-prc.example.com:911
```

## 配置wget代理
/etc/wgetrc 或 ~/.wgetrc
```
http_proxy=http://proxy-prc.example.com:911
https_proxy=http://proxy-prc.example.com:911
ftp_proxy=http://proxy-prc.example.com:911
```

## 关闭防火墙
```bash
sudo systemctl disable firewalld.service
```


## 自动登录
修改 /etc/gdm/custom.conf
```ini
[daemon]
AutomaticLogin=username
AutomaticLoginEnable=true
TimedLoginEnable=true
TimedLogin=username
```

## 设置DNS服务器
### 显示当前网络连接

一个连接(connection)就是/etc/sysconfig/network-scripts/目录下的一个配置文件，  
接口(device)是物理设备，一个物理设备可以拥有多个配置文件，  
但只能有一个配置文件属于使用(active)状态；配置文件的生成与使用状态均由NetworkManager控制。 参考于[这里](https://www.jianshu.com/p/5d5560e9e26a)

启动和停止接口
```sh
nmcli connection down connection-name
nmcli connection up connection-name
nmcli device disconnect interface-name
nmcli device connect interface-name
```

```sh
[media@s5n8c2 ~]$ nmcli connection show
NAME      UUID                                  TYPE            DEVICE
docker0   0138153f-46e6-4157-9e3b-75c7246bd66a  bridge          docker0
enp1s0f0  6bad5725-2baa-4db0-b022-4805c7b49603  802-3-ethernet  enp1s0f0
virbr0    e32166a5-96bd-41a9-a5ca-eaaaac23d3a4  bridge          virbr0
enp1s0f1  04f7ac81-be60-4cfd-a996-8cb2f81e894e  802-3-ethernet  --
enp4s0    6c15bcdb-e9a5-4b97-8120-9575a779fd3f  802-3-ethernet  --

[media@s5n8c2 ~]$ sudo nmcli -t connection up enp4s0
Connection successfully activated (D-Bus active path:
/org/freedesktop/NetworkManager/ActiveConnection/5)

[media@s5n8c2 ~]$ sudo nmcli -t connection down enp4s0
Connection 'enp4s0' successfully deactivated (D-Bus active path: 
/org/freedesktop/NetworkManager/ActiveConnection/5)
```

创建连接
```sh
nmcli connection add type ethernet con-name connection-name ifname interface-name
nmcli connection add type ethernet con-name connection-name ifname interface-name ip4 address gw4 address
```

```sh
[media@s5n8c2 ~]$ sudo nmcli connection add type ethernet con-name myeth ifname enp1s0f1
Connection 'myeth' (b95f4b6e-4024-4687-a0b0-9fcfdb4c1d36) successfully added.

[media@s5n8c2 ~]$ nmcli connection show myeth
```

```bash
#修改当前网络连接对应的DNS服务器，这里的网络连接可以用名称或者UUID来标识
sudo nmcli con mod enp117s0 ipv4.dns "10.248.2.5 10.239.27.228"

#将dns配置生效
sudo nmcli con up enp117s0
```


```bash
# set hostname
[root@localhost ~]# hostnamectl set-hostname dlp.srv.world
# display devices
[root@localhost ~]# nmcli d 
DEVICE       TYPE      STATE      CONNECTION
eth0         ethernet  connected  eth0
lo           loopback  unmanaged  --

# set IPv4 address ⇒ nmcli *** [IP address]
[root@localhost ~]# nmcli c modify eth0 ipv4.addresses 10.0.0.30/24 
# set default gateway
[root@localhost ~]# nmcli c modify eth0 ipv4.gateway 10.0.0.1 
# set DNS
[root@localhost ~]# nmcli c modify eth0 ipv4.dns 10.0.0.1 
# set manual for static setting (it's "auto" for DHCP)
[root@localhost ~]# nmcli c modify eth0 ipv4.method manual 
# restart the interface and reload the settings
[root@localhost ~]# nmcli c down eth0; nmcli c up eth0 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)
# show settings
[root@localhost ~]# nmcli d show eth0 
GENERAL.DEVICE:                 eth0
GENERAL.TYPE:                   ethernet
GENERAL.HWADDR:                 00:0C:29:CD:9C:2D
GENERAL.MTU:                    1500
GENERAL.STATE:                  100 (connected)
GENERAL.CONNECTION:             eth0
GENERAL.CON-PATH:               /org/freedesktop/NetworkManager/ActiveConnection/0
WIRED-PROPERTIES.CARRIER:       on
IP4.ADDRESS[1]:                 ip = 10.0.0.30/24, gw = 10.0.0.1
IP4.DNS[1]:                     10.0.0.1
IP6.ADDRESS[1]:                 ip = fe80::20c:29ff:fecd:9c2d/64, gw = ::

# show status
[root@localhost ~]# ip addr show 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:cd:9c:2d brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fecd:9c2d/64 scope link
       valid_lft forever preferred_lft forever
```



## 修改启动方式
[systemd](https://wiki.archlinux.org/index.php/systemd)

```bash
sudo systemctl get-default                          # 查看当前 启动方式
sudo systemctl set-default -f multi-user.target     #  runlevel 3
sudo systemctl set-default -f graphical.target      #  runlevel 5

sudo systemctl isolate multi-user.target            # 在不重新启动下 进入文本模式
```

## 生成 grub.cfg 文件
```sh
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 进入CentOS7的救援模式
开机在 kernel 选项后面加上 `systemd.unit=rescue.target` 然后 `Ctrl + X` 进入救援模式。


## 进入CentOS7的多用户模式
开机在 kernel 选项后面加上 `systemd.unit=multi-user.target` 然后 `Ctrl + X` 进入救援模式。

## 修改CentOS7 root密码
开机 选择kernel  在kernel选项中 去掉 rhgb和 quiet。 在后面添加 `init=/bin/sh` 按`Ctrl + X` 进入系统

```sh
mount -o remount,rw /
passwd root

touch /.autorelabel
exec /sbin/init
```

以下来自鸟哥的方法
开机 选择kernel  在kernel选项在最后面添加 `rd.break` 按`Ctrl + X` 进入系统

```sh
mount | grep sysroot
mount -o remount,rw /sysroot
chroot /sysroot

passwd root

touch /.autorelabel
exit
reboot
```

- `rd.break` 是进入initramfs的RAM Disk环境下
- remount 是把真实的系统mount上去, 并且给与读写权限
- chroot 改变根目录
- `touch ./autorelabel` 因为在RAM Disk环境下没有SELinux，我们修改密码了，也就是修改了/etc/shadow, SELinux安全本文特性就会被取消，有了`/.autorelabel`, 系统重启之后会重新写入SELinux安全本文到每一个文件中。 如果关闭了SELInux这一步可以省略。

> 在CentOS6 中，只要在kernel选项中 `single` 进入系统后就能修改。

## systemctl
| Sysvinit(RHEL6) | Systemctl(RHEL7) | 说明     |
| :---------      | :--------        | :------- |
|`service foo start`|`systemctl start foo.service`| 启动服务|
|`service foo restart`|`systemctl restart foo.service`| 重启服务|
|`service foo stop`|`systemctl stop foo.service`| 停止服务|
|`service foo reload`|`systemctl reload foo.service`| 重新加载配置文件(不终止服务)|
|`service foo status`|`systemctl status foo.service`| 查看服务状态|
|`chkconfig foo on`|`systemctl enable foo.service`|设置开机自动启动|
|`chkconfig foo off`|`systemctl disable foo.service`|设置开机不自动启动|
|`chkconfig foo` |`systemctl is-enabled foo.service`|查看特定服务是否开机启动|
|`chkconfig --list` |`systemctl list-unit-files --type=service`|查看各个级别下服务的启动与禁用情况|

/usr/lib/systemd/system: 默认启动脚本在这里，到/etc/systemd/system下修改

/run/systemd/system: 系统执行过程中所产生的服务脚本， 优先级比 /usr/lib/systemd/system 高

/etc/systemd/system: 软连接， 执行顺序比/run/systemd/system 高

/etc/sysconfig/*: 几乎所有的服务将初始化的一些选项写在这个目录下

## NFS
### install nfs
```bash
sudo yum install -y nfs-server nfs-utils
sudo systemctl start nfs-server.service
```

### config
vim /etc/exports
```
/home/user/nfs/ *(rw,sync)
```

### mount on client
```bash
showmount -e [IP]
sudo mount -t nfs -o vers=3 [IP]:/home/user/nfs /media
```

## 卸载软件 但不卸载依赖
```bash
rpm -qa | grep gstreamer  
sudo rpm -e --nodeps gstreamer
```

## 修改时区
```bash
sudo date -s "2016/08/17 14:37:00"
sudo hwclock -w
sudo timedatectl set-timezone Asia/Shanghai
```



## rpm  安装包
```bash
# 安装一个rpm包
rpm -ivh filename

# 升级一个rpm包
rpm -Uvh filename

# 查询
rpm -q filename

# 查询包具体信息
rpm -qi libjpeg-turbo-devel

# 查询包里内容
rpm -qip xxx.rpm

# 查询全部
rpm -qa | grep filename

# 卸载软件 但不卸载依赖
rpm -qa | grep gstreamer  
sudo rpm -e --nodeps gstreamer

# 列出某一个文件属于哪个rpm包
rpm -qf /usr/lib/libjpeg.so

# 解压包内容
rpm2cpio xxx.rpm | cpio -div
```


## yum
```bash
# 搜索
yum search vim
yum list |grep 'vim'

# 查看已经安装的
yum list installed | grep kernel

# 安装
yum install vim-X11

# 卸载
yum remove vim-X11

# 升级rpm包
yum update libselinux

# 只下载到指定目录 不安装
yum install -y yum-presto.noarch  --downloadonly --downloaddir=/usr/local/src/

# 下载软件包
yumdownloader gcc

# 查找组
yum grouplist
# 安装组
yum groupinstall 'Development Tools'

## 安装指定系统中的
yum install --releasever=7.4.1708 gcc
```

yum 安装指定版本
```sh
[media@centos7-vm centos]$ yum list kernel
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Installed Packages
kernel.x86_64                      3.10.0-693.el7                         @anaconda
kernel.x86_64                      3.10.0-693.11.6.el7                    installed
kernel.x86_64                      3.10.0-693.17.1.el7                    installed
kernel.x86_64                      3.10.0-693.21.1.el7                    installed
Available Packages
kernel.x86_64                      3.10.0-862.3.2.el7                     updates

yum install -y 3.10.0-862.3.2.el7
```


|阶段|System|Kernel|
|:---|:----|:----|
|before|CentOS5.5|2.6.18-194.el5|
|yum upgrade|CentOS5.7|2.6.18-194.el5|
|yum update|CentOS5.7|2.6.18-238.el5|

> yum -y update
> 升级所有包，改变软件设置和系统设置,系统版本内核都升级

> yum -y upgrade
> 升级所有包，不改变软件设置和系统设置，系统版本升级，内核不改变

## 修改 /etc/default/grub 并生效

```
[media@s5n5c1 ~]$ sudo cat /etc/default/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet console=ttyS0,115200n8"
GRUB_DISABLE_RECOVERY="true"
```

生成grub配置文件
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg 
```

> GRUB_DEFAULT=saved 修改为 GRUB_DEFAULT=0  可以从第一个内核启动

### 修改grub默认启动项

/boot/grub2/grub.cfg
```
menuentry 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.el7.x86_64-advanced-9b75fa52-f76a-4b1b-b004-e9e5f60354fb'
```

```shell
sudo grub2-set-default 'CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)'

sudo grub2-editenv list
```



