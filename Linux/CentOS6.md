# CentOS6

[TOC]

## 配置yum 代理
/etc/yum.conf
```
proxy=http://proxy-prc.example.com:911
```

## 配置yum本地源
/etc/yum.repos.d/CentOS-Media.repo
```
# CentOS-Media.repo
#
#  This repo can be used with mounted DVD media, verify the mount point for
#  CentOS-6.  You can use this repo and yum to install items directly off the
#  DVD ISO that we release.
#
# To use this repo, put in your DVD and use it with the other repos too:
#  yum --enablerepo=c6-media [command]
#
# or for ONLY the media repo, do this:
#
#  yum --disablerepo=\* --enablerepo=c6-media [command]

[c6-media]
name=CentOS-$releasever - Media
baseurl=file:///media/CentOS/
        file:///media/cdrom/
        file:///media/cdrecorder/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
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
chkconfig --level 35 iptables off
chkconfig --level 35 ip6tables off
```

## 添加sudo权限
/etc/sudoers
```
root   ALL=(ALL)       ALL
media   ALL=(ALL)       ALL
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

## yum 安装软件
```bash
sudo yum install -y gcc cmake gcc-c++ automake libtool libdrm-devel
sudo yum install -y libX11-devel libpciaccess-devel xorg-x11-server-devel
sudo yum install -y ncurses ncurses-devel flex cmake bison
sudo yum install -y git zlib zlib-devel tk tk-devel

sudo yum groupinstall -y "Development Tools"
```


## 升级kernel
```bash
mkdir ~/kernel_install
cd ~/kernel_install
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.14.5.tar.gz
tar xf linux-3.14.5.tar.gz
cd linux-3.14.5

sudo make olddefconfig
sudo make -j8
sudo make modules_install
sudo make install
reboot   # reboot to new kernel
```



