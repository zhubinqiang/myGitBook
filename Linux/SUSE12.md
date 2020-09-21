# SUSE12

[TOC]

## 添加zypper repo
```bash
sudo zypper ar -t yast2 iso:/?iso=SLES-11-SP3-DVD-x86_64-GM-DVD1.iso SLES-11-SP3-DVD-x86_64-GM-DVD1

sudo zypper ar -t yast2 iso:/?iso=SLE-11-SP3-SDK-DVD-x86_64-Beta3-DVD1.iso SLE-11-SP3-SDK-DVD-x86_64-Beta3-DVD1

sudo zypper refresh
```


## 安装软件
```bash
sudo zypper in -y git gitk gcc gcc-c++ automake
sudo zypper in -y flex cmake bison perl-Archive-Zip libtool
sudo zypper in -y xorg-x11-devel libdrm-devel libpciaccess0-devel
sudo zypper in -y libreoffice
```


