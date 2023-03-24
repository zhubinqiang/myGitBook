# Ubuntu14
[TOC]

## Ubuntu 14.04 用aliyun 的源
/etc/apt/sources.list

```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

## 设置apt-get代理
/etc/apt/apt.conf
```
Acquire::http::proxy http://proxy-prc.example.com:911;
Acquire::https::proxy http://proxy-prc.example.com:911;
Acquire::ftp::proxy "ftp://proxy-prc.example.com:911/";
Acquire::http::proxy::linux-ftp.sh.example.com "DIRECT";
```

碰到有用户名和密码

```
Acquire::http::proxy "http://user:pwd@proxy.xxx:911";
```

> 但是密码中使用了@！等特殊字符，就要小心了。普通的\@转义在这里无效。正确的做法是用%+ASCII的十六进制码

```
@==>%40
$==>%24
!==>%21
```

## apt
### apt-get
```sh
## 更新软件列表
apt-get update

#更新所有已安装的软件包
apt-get upgrade

## 将系统升级到新版本
apt-get dist-upgrade 

## 安装
apt-get install python-apt

## 删除软件 但保留配置
apt-get remove python-apt

## 删除软件和配置
apt-get remove --purge python-apt
```

### apt-cache
```sh
## 搜索包
apt-cache search python-apt

##(package 获取包的相关信息，如说明、大小、版本等)
apt-cache show python-apt
```

## dpkg

```sh
## 安装
dpkg -i mozybackup_i386.deb

## 列出包内容
dpkg -c mozybackup_i386.deb

## 解压包
dpkg –unpack mozybackup_i386.deb

## 查询是否安装
dpkg -l | grep python-apt

## 查看某个文件的归属包
dpkg -S python-apt

## 查看安装版本
dpkg -l python-apt

## 查询详细详细
dpkg -s python-apt

## 查看软件安装到什么地方
dpkg -L python-apt

## 删除软件 但保留配置
dpkg -r mozybackup

## 删除软件和配置
dpkg -P mozybackup
```

## dpkg 打deb包
```
mydeb
├── DEBIAN
│   ├── control
│   ├── postinst
│   └── postrm
└── usr
    └── bin
        └── mytest
```

mydeb/DEBIAN/control
```
Package: mydeb
Version: 1.4.0-2019.06.04
Section: devel
Priority: optional
Depends: libc6 (>= 2.17), libgcc1 (>= 1:3.0), libstdc++6 (>= 5.2)
Suggests:
Architecture: amd64
Installed-Size: 4096
Maintainer: whoami
Provides: bioinfoserv-arb
Description: a demo to make a deb package
```

mydeb/DEBIAN/postinst  和 postrm都是
```
/sbin/ldconfig
```

> 注意给 postinst 和 postrm 可执行权限

打包
```sh
dpkg-deb -b mydeb mydeb-1.4.0-2019.06.04-amd64.deb
```


```sh
## 解压出控制信息
dpkg-deb -e mydeb-1.4.0-2019.06.04-amd64.deb

## 解压文件
dpkg-deb -X ../mydeb-1.4.0-2019.06.04-amd64.deb .
```


## 设置全局代理
/etc/environment
```
export http_proxy=http://proxy-prc.example.com:911
export https_proxy=http://proxy-prc.example.com:911
```

## git-proxy
```shell
sudo apt-get install socat
```

vim /usr/bin/git-proxy
```
proxy=proxy-shz.example.com
exec socat STDIO SOCKS4:$proxy:$1:$2
```

```bash
sudo chmod a+x /usr/bin/git-proxy
```

~/.gitconfig
```
[core]
        gitproxy = none for example.com
        gitproxy = git-proxy
[http]
        proxy = http://proxy-prc.example.com:911
        sslverify = false
[https]
        proxy = http://proxy-prc.example.com:911
```

~/.gitconfig
```
[core]
        gitproxy = none for example.com
        gitproxy = git-proxy
[http]
        proxy = http://name:password@child-prc.example.com:914
        sslverify = false
[https]
        proxy = http://name:password@child-prc.example.com:914
```


## 为x11vnc 配置开机启动
/etc/init/x11vnc.conf
```
start on login-session-start
script
    x11vnc -display :0 -auth /var/run/lightdm/root/:0 -forever -bg -o /var/log/x11vnc.log -rfbport 5900
end script
```

## 网卡配置
/etc/network/interfaces
```
auto eth0
iface eth0 inet static 
#   hwaddress ether 00:1e:67:99:6c:ef
    address 192.168.1.11
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-search example.com
    dns-nameservers 10.248.2.1 10.239.27.228 172.17.6.9

auto eth1
iface eth1 inet static 
#   hwaddress ether 00:1e:67:99:6c:ee
    address 10.0.0.11
    netmask 255.255.255.0

# disable eth2
iface eth2 inet manual
```

```sh
sudo ifup eth0
sudo ifup eth1
```

netplan的方式
/etc/netplan/01-network-manager-all.yaml

配置静态IP
```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eno1:
      addresses: [192.168.1.11/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [10.248.2.1,10.239.27.228]
      dhcp4: no
```

动态IP
```yaml
network:
  ethernets:
    eno1:
      dhcp4: true
      optional: true
    eno2:
      dhcp4: true
      optional: true
  version: 2
```
> 上面的 **optional: true** 是为了 避免开机的时候 A start job is running for wait for network to be configured.
> 参考这里[askubuntu](https://askubuntu.com/questions/972215/a-start-job-is-running-for-wait-for-network-to-be-configured-ubuntu-server-17-1/1110474#1110474)


```sh
sudo netplan --debug apply
```

在kernel的命令行添加 `net.ifnames=0` 可以修改成之前的网卡名称像eth0 参考[这里](https://blog.csdn.net/lirui1212/article/details/105180751)

参考 https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/
1. Names incorporating Firmware/BIOS provided index numbers for on-board devices (example: eno1)
2. Names incorporating Firmware/BIOS provided PCI Express hotplug slot index numbers (example: ens1)
3. Names incorporating physical/geographical location of the connector of the hardware (example: enp2s0)
4. Names incorporating the interfaces's MAC address (example: enx78e7d1ea46da)
5. Classic, unpredictable kernel-native ethX naming (example: eth0)


## 设置DNS
DNS 在/etc/resolv.conf中设置不起效
```sh
cat /etc/resolvconf/resolv.conf.d/tail
nameserver 10.248.2.5
nameserver 10.239.27.228
```

显示 dns
```sh
nm-tool
```

## 服务命令
|任务|default|安装sysvconfig和sysv-rc-conf|
|:---|:---|:---|
|启动服务|/etc/init.d/apache start|service apache start|
|停止服务|/etc/init.d/apache stop|service apache stop|
|服务开机自启动|update-rc.d apache defaults|sysv-rc-conf apache on|
|禁止开机自启动|update-re.d apache purge|sysv-rc-conf apache off|

## wireshark 抓包工具
```bash
sudo apt-get install -y wireshark
sudo dpkg-reconfigure wireshark-common
sudo wireshark
```

## 14.04.x ubuntu kernel support schedule
refer to [https://wiki.ubuntu.com/Kernel/RollingLTSEnablementStack](https://wiki.ubuntu.com/Kernel/RollingLTSEnablementStack)

```
2014/04                                                          2019/04
+----------------------------------------------------------------------+
|  14.04.0(v3.13)                                                   5y |
+----------------------------------------------------------------------+
v3.13 == Trusty 14.04 GA kernel

   2014/08
   +-------------------------------------------------------------------+
   | 14.04.1(v3.13)                                              4y 9m |
   +-------------------------------------------------------------------+
    v3.13 == Trusty 14.04 GA kernel

       2015/02
       +----------------------------------+
       |   14.04.2(v3.16)             18m |
       +----------------------------------+
        v3.16 == Utopic 14.10 HWE Kernel

               2015/08
               +--------------------------+
               | 14.04.3(v3.19)       12m |
               +--------------------------+
                v3.19 == Vivid 15.04 HWE Kernel

                       2016/02
                       +------------------+
                       | 14.04.4(v4.2) 6m |
                       +------------------+
                       v4.2 == Wily 15.10 HWE kernel

                                          2016/08
                                          +---------------------------+
                                          |14.04.5(v4.4)        2y 9m |
                                          +---------------------------+
                                          v4.4 = Xenial 16.04 HWE kernel
```

## vncserver
```
sudo apt-get install vnc4server
```

```
media@CBX:~/.vnc$ cat xstartup
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
```

Unity风格
```sh
sudo apt-get install --no-install-recommends ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```

开机启动VNCSERVER
```sh
sudo vim /etc/systemd/system/vncserver@.service
```

```
[Unit]

Description=Start TightVNC server at startup

After=syslog.target network.target

[Service]

Type=forking

User=yourusername

PAMName=loginPIDFile=/home/yourusername/.vnc/%H:%i.pid

ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1

ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i

ExecStop=/usr/bin/vncserver -kill :%i[Install]WantedBy=multi-user.target
```


```sh
sudo systemctl daemon-reload

sudo systemctl enable vncserver@1.service

sudo systemctl start vncserver@1

sudo systemctl status vncserver@1
```

## 修改系统语言
```
sudo dpkg-reconfigure locales
```

## 修改grub启动顺序

先查询有多少个启动项 可以通过 `grep menuentry /boot/grub/grub.cfg` 大致如下的内容

```
menuentry 'Ubuntu'
submenu 'Advanced options for Ubuntu' 
        menuentry 'Ubuntu, with Linux 4.4.0-21-generic'
        menuentry 'Ubuntu, with Linux 4.4.0-21-generic (recovery mode)'
        menuentry 'Ubuntu, with Linux 4.4.0-21-generic (upstart)'
menuentry 'Memory test (memtest86+)'
menuentry 'Memory test (memtest86+, serial console 115200)'
```

再修改 /etc/default/grub 中的 **GRUB_DEFAULT**

```
GRUB_DEFAULT="1>2"
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```

- 启动顺序从 **0** 开始计算, 此列中0是 'Ubuntu' 的那个menuentry
- 上面的 **"1>2"** submenuentry中为2，此列中是 upstart 那一项
- 通过注释掉这行 **#GRUB_HIDDEN_TIMEOUT=0** 开机有grub 菜单

最后再执行 `update-grub2` 更新grub

## 忘记root密码
先进入grub, 在kernel选项后面加删除`ro  quiet splash $vt_handoff` 再添上 `rw init=/bin/bash`
```
linux   /boot/vmlinuz-5.3.7 root=UUID=adad0ceb-816d-475f-a259-3bef7504723c ro  quiet splash $vt_handoff
```

如下：
```
linux   /boot/vmlinuz-5.3.7 root=UUID=adad0ceb-816d-475f-a259-3bef7504723c rw init=/bin/bash 
```

按 **Ctrl + X** 进入系统

进系统后, 输入如下，进行修改密码。然后再重启。
```sh
passwd root
```

## FAQ
A:waiting up to 60 more seconds for network configuration
Q: 在 /etc/init/failsafe.conf 注释掉 `sleep 59`
```
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :
sleep 59
```


