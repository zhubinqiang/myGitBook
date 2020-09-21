# 在Ubuntu 14.04上配置 PXE 服务器
[TOC]

主要来源于[这里](https://www.maketecheasier.com/configure-pxe-server-ubuntu/)

## 安装流程
[安装流程摘录于这里](https://blog.csdn.net/trochiluses/article/details/11736119)

### 安装流程
1. 客户机从自己的PXE网卡启动，**向本网络中的DHCP服务器索取IP，并搜寻引导文件的位置**
2. DHCP服务器返回分给**客户机IP以及bootstrap文件的放置位置**(该文件一般是放在一台TFTP服务器上)
3. 客户机向本网络中的TFTP服务器**索取bootstrap文件**
4. 客户机取得bootstrap文件之后**执行该文件**
5. 根据bootstrap的执行结果，**通过TFTP服务器加载内核和文件系统**
6. 进入安装画面, 此时可以通过选择FTP,HTTP,NFS方式之一进行安装

![](images/PXE/pxe-flow.jpg)

### 流程小结
1. client的网卡寻找dhcp服务器，由服务器上/etc/dhcp.conf得到ip和引导程序所在地址
2. 服务器上有dhcp.conf（引导程序文件名）和tftp的配置（tftp跟路径，里面有引导程序和系统初始化程序），client得到引导程序pxelinux启动文件的绝对路径，运行引导程序，读取启动配置文件pxelinux.cfg/default，得到操作系统初始化的两个相关文件vmlinuz, initrd.img
3. 选择安装方式 
客户端广播dhcp请求——服务器相应请求，建立链接——由dhcp和tftp配置得到ip还有引导程序所在地点——客户端下载引导程序并开始运行——引导程序读取

### 相关文件位置与内容
dhcp 配置文件`/etc/dhcpd/dhcp.conf` ip管理与引导程序名称
tftp 配置文件`/etc/xinetd.d/tftp` tftp根目录，和上面的引导程序名称组成完整路径
引导程序读取的配置文件`/var/lib/tftpboot/tftpboot/pxelinux.cfg/default` 启动内核其他

## 预备
1. 安装好 Ubuntu 14.04 系统的主机
2. 支持DHCP的路由器一个
3. 支持pxe启动的目标机器
4. 主机和目标机器已经通过路由器链接起来了

## 配置网络

1.修改 /etc/network/interfaces 文件
```
# The loopback network interface
auto lo
iface lo inet loopback
# The primary network interface
auto eth0
iface eth0 inet static
address 192.168.1.20
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 8.8.8.8
```

重启网络
```bash
sudo /etc/init.d/networking restart
```

> 这一步未必起效。ifconfig 命令可能ip没有改变， 可以从 System Settings –> Network –> 去设置静态IP
> 由于配置静态IP了，可能会导致无法连接外网。配置网络可以等到apt-get install的步骤之后来配置。或者Ubuntu的机器配置双网卡，一个连接外网，另一个连接交换机。


## 安装 DHCP，TFTP，NFS:

```bash
sudo apt-get update
sudo apt-get install isc-dhcp-Server inetutils-inetd tftpd-hpa syslinux nfs-kernel-Server
```

## 配置 DHCP:
修改 “/etc/default/dhcp-server”
```
INTERFACES="eth0"
```

修改 /etc/dhcp/dhcpd.conf
```
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.21 192.168.1.240;
option subnet-mask 255.255.255.0;
option routers 192.168.1.20;
option broadcast-address 192.168.1.255;
filename "pxelinux.0";
next-Server 192.168.1.20;
}
```

- **next-Server 192.168.1.20**: 指定PXE的服务器 IP是192.168.1.20
- **filename "pxelinux.0"**: 指定PXE引导程序。这里是相对路径，其目录在后面的TFTP 和 inetd中配置。


启动 DHCP service.
```bash
sudo /etc/init.d/isc-dhcp-server start
```

## 配置 TFTP:
TFTP: Trivial File Transfer Protocol (简单文件传输协议)

修改 /etc/inetd.conf
```
tftp dgram udp wait root /usr/sbin/in.tftpd /usr/sbin/in.tftpd -s /var/lib/tftpboot
```


修改 /etc/default/tftpd-hpa
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS="[:0.0.0.0:]:69"
TFTP_OPTIONS="--secure"
RUN_DAEMON="yes"
OPTIONS="-l -s /var/lib/tftpboot"
```

设置开机启动
```bash
sudo update-inetd --enable BOOT
sudo service tftpd-hpa start
```

检查状态
```bash
sudo netstat -lu
```

有如下输出
```
Proto Recv-Q Send-Q Local Address Foreign Address State
udp 0 0 *:tftp *:*
```

## 配置pxe 启动文件

```bash
sudo mkdir /var/lib/tftpboot
sudo mkdir /var/lib/tftpboot/pxelinux.cfg
sudo mkdir -p /var/lib/tftpboot/Ubuntu/14.04/amd64/
sudo cp /usr/lib/syslinux/vesamenu.c32 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/pxelinux.0 /var/lib/tftpboot/
```

## 配置pxe 配置文件
修改 /var/lib/tftpboot/pxelinux.cfg/default
```
DEFAULT vesamenu.c32
TIMEOUT 100
PROMPT 0
MENU INCLUDE pxelinux.cfg/PXE.conf
NOESCAPE 1


LABEL local
MENU LABEL Boot from local driver
MENU DEFAULT
localboot 0


LABEL Install Auto Ubuntu 14.04 Desktop
MENU LABEL Auto Install Ubuntu 14.04 Desktop
kernel Ubuntu/14.04/vmlinuz.efi
append boot=casper automatic-ubiquity netboot=nfs nfsroot=192.168.1.20:/var/lib/tftpboot/Ubuntu/14.04/amd64 initrd=Ubuntu/14.04/initrd.lz quiet splash url=http://192.168.1.20/ubuntu/14.04/preseed.cfg  DEBCONF_DEBUG=5 vga=normal
ENDTEXT


LABEL Install Auto Ubuntu 14.04.4 Desktop
MENU LABEL Auto Install Ubuntu 14.04.4 Desktop
kernel Ubuntu/14.04.4/vmlinuz.efi
append boot=casper automatic-ubiquity netboot=nfs nfsroot=192.168.1.20:/var/lib/tftpboot/Ubuntu/14.04.4/amd64 initrd=Ubuntu/14.04.4/initrd.lz quiet splash url=http://192.168.1.20/ubuntu/14.04.4/preseed.cfg  DEBCONF_DEBUG=5 vga=normal
ENDTEXT


LABEL Install Auto Ubuntu 14.04.5 Desktop
MENU LABEL Auto Install Ubuntu 14.04.5 Desktop
kernel Ubuntu/14.04.5/vmlinuz.efi
append boot=casper automatic-ubiquity netboot=nfs nfsroot=192.168.1.20:/var/lib/tftpboot/Ubuntu/14.04.5/amd64 initrd=Ubuntu/14.04.5/initrd.lz quiet splash url=http://192.168.1.20/ubuntu/14.04.5/preseed.cfg  DEBCONF_DEBUG=5 vga=normal
ENDTEXT


LABEL Install Auto Ubuntu 16.04 Desktop
MENU LABEL Auto Install Ubuntu 16.04 Desktop
kernel Ubuntu/16.04/vmlinuz.efi
append boot=casper automatic-ubiquity netboot=nfs nfsroot=192.168.1.20:/var/lib/tftpboot/Ubuntu/16.04/amd64 initrd=Ubuntu/16.04/initrd.lz quiet splash url=http://192.168.1.20/ubuntu/16.04/preseed.cfg  DEBCONF_DEBUG=5 vga=normal
ENDTEXT


LABEL Install Auto Ubuntu 16.10 Desktop
MENU LABEL Auto Install Ubuntu 16.10 Desktop
kernel Ubuntu/16.10/vmlinuz.efi
append boot=casper automatic-ubiquity netboot=nfs nfsroot=192.168.1.20:/var/lib/tftpboot/Ubuntu/16.10/amd64 initrd=Ubuntu/16.10/initrd.lz quiet splash url=http://192.168.1.20/ubuntu/16.10/preseed.cfg  DEBCONF_DEBUG=5 vga=normal
ENDTEXT


LABEL Install CentOS6.4
MENU LABEL Auto Install CentOS 6.4
kernel CentOS/6.4/vmlinuz
append initrd=CentOS/6.4/initrd.img method=http://192.168.1.20/CentOS/6.4/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/6.4/auto.ks
ENDTEXT


LABEL Install CentOS6.5
MENU LABEL Auto Install CentOS 6.5
kernel CentOS/6.5/vmlinuz
append initrd=CentOS/6.5/initrd.img method=http://192.168.1.20/CentOS/6.5/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/6.5/auto.ks
ENDTEXT


LABEL Install CentOS6.6
MENU LABEL Auto Install CentOS 6.6
kernel CentOS/6.6/vmlinuz
append initrd=CentOS/6.6/initrd.img method=http://192.168.1.20/CentOS/6.6/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/6.6/auto.ks
ENDTEXT


LABEL Install CentOS6.7
MENU LABEL Auto Install CentOS 6.7
kernel CentOS/6.7/vmlinuz
append initrd=CentOS/6.4/initrd.img method=http://192.168.1.20/CentOS/6.7/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/6.7/auto.ks
ENDTEXT


LABEL Install CentOS6.8
MENU LABEL Auto Install CentOS 6.8
kernel CentOS/6.8/vmlinuz
append initrd=CentOS/6.8/initrd.img method=http://192.168.1.20/CentOS/6.8/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/6.8/auto.ks
ENDTEXT


LABEL Install CentOS7.0
MENU LABEL Auto Install CentOS 7.0
kernel CentOS/7.0/vmlinuz
append initrd=CentOS/7.0/initrd.img method=http://192.168.1.20/CentOS/7.0/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/7.0/auto.ks
ENDTEXT


LABEL Install CentOS7.1
MENU LABEL Auto Install CentOS 7.1
kernel CentOS/7.1/vmlinuz
append initrd=CentOS/7.1/initrd.img method=http://192.168.1.20/CentOS/7.1/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/7.1/auto.ks
ENDTEXT


LABEL Install CentOS7.2
MENU LABEL Auto Install CentOS 7.2
kernel CentOS/7.2/vmlinuz
append initrd=CentOS/7.2/initrd.img method=http://192.168.1.20/CentOS/7.2/x86_64 devfs=nomount ks=http://192.168.1.20/CentOS/7.2/auto.ks
ENDTEXT


LABEL auto SUSE12 SP2 GM
MENU LABEL auto SUSE12 SP2 GM
kernel SUSE/SP2_12_GM/linux
append initrd=SUSE/SP2_12_GM/initrd install=http://192.168.1.20/SP2_12_GM/x86_64 autoyast=http://192.168.1.20/SP2_12_GM/autoinst.xml splash=verbose showopts
ENDTEXT


LABEL Install Auto test Debian8.6
MENU LABEL Auto Install Debian 8.6 
kernel Debian/8.6/linux 
append netcfg/choose_interface=auto initrd=Debian/8.6/initrd.gz vga=normal
ENDTEXT
```

### CentOS
先把iso镜像文件mount一下, 以centos7.1为例
```
sudo mount -o loop CentOS-7-x86_64-DVD-1503-01.iso /mnt
sudo mkdir -p /var/lib/tftpboot/CentOS/7.1/x86_64
sudo cp /mnt/isolinux/vmlinuz /var/lib/tftpboot/CentOS/7.1/
sudo cp /mnt/isolinux/initrd.img /var/lib/tftpboot/CentOS/7.1/
sudo cp -air /mnt/* /var/lib/tftpboot/CentOS/7.1/x86_64
```

### Ubuntu
以ubuntu14.04.5为例
```bash
sudo mount -o loop ubuntu-14.04.5-desktop-amd64.iso /mnt
sudo mkdir -p /var/lib/tftpboot/Ubuntu/14.04.5/amd64
sudo cp /mnt/casper/vmlinuz.efi /var/lib/tftpboot/Ubuntu/14.04.5/
sudo cp /mnt/casper/initrd.lz /var/lib/tftpboot/Ubuntu/14.04.5/
sudo cp -air /mnt/* /var/lib/tftpboot/Ubuntu/14.04.5/amd64
```

### SUSE
boot/x86_64/loader/
```bash
sudo mount -o loop SLE-12-SP2-Desktop-DVD-x86_64-GM-DVD1.iso /mnt
sudo mkdir -p /var/lib/tftpboot/SUSE/SP2_12_GM/x86_64/
sudo cp /mnt/boot/x86_64/loader/initrd /var/lib/tftpboot/SUSE/SP2_12_GM/
sudo cp /mnt/boot/x86_64/loader/linux /var/lib/tftpboot/SUSE/SP2_12_GM/
sudo cp -air /mnt/* /var/lib/tftpboot/SUSE/SP2_12_GM/x86_64/
```


/var/lib/tftpboot/pxelinux.cfg/pxe.conf :
```
MENU TITLE PXE Server
NOESCAPE 1
ALLOWOPTIONS 1
PROMPT 0
MENU WIDTH 80
MENU ROWS 14
MENU TABMSGROW 24
MENU MARGIN 10
MENU COLOR border 30;44 #ffffffff #00000000 std
```

## 配置 nfs
/etc/exports
```
/var/lib/tftpboot/Ubuntu *(ro,async,no_root_squash,no_subtree_check)
/var/lib/tftpboot/CentOS *(ro,async,no_root_squash,no_subtree_check)
/var/lib/tftpboot/Debian *(ro,async,no_root_squash,no_subtree_check)
/var/lib/tftpboot/Windows *(ro,async,no_root_squash,no_subtree_check)
```

重启 nfs
```bash
sudo /etc/init.d/nfs-kernel-server start
```

## 配置apache
/etc/apache2/apache2.conf
```
Alias /ubuntu "/var/lib/tftpboot/Ubuntu"
<Directory /var/lib/tftpboot/Ubuntu/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

Alias /suse "/var/lib/tftpboot/SUSE"
<Directory /var/lib/tftpboot/SUSE/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

Alias /debian "/var/lib/tftpboot/Debian"
<Directory /var/lib/tftpboot/Debian/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

Alias /CentOS "/var/lib/tftpboot/CentOS/"
<Directory /var/lib/tftpboot/CentOS/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```


## FAQ
```
sudo vim /etc/initramfs-tools/initramfs.conf
```

```
DEVICE=         # DEVICE=eth0
```


## Issues
1. Ubuntu14.04 安装过程中图像模糊
2. Ubuntu16.04 安装完之后不能调出终端
3. 有时候 CentOS 需要点击 "OK" 在遇到 "unsupported hardware" 
4. SUSE12 在重启之后需要自己去配置。


## Appendix

### debian
1. [initrd.gz](http://mirrors.163.com//debian/dists/jessie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz)
2. [linux](http://mirrors.163.com//debian/dists/jessie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux)
3. [DVD iso](http://mirrors.163.com//debian-cd/8.6.0/amd64/iso-dvd/debian-8.6.0-amd64-DVD-1.iso)


DVD iso
```bash
mkdir dvd temp
mount -o loop debian-8.6.0-amd64-DVD-1.iso dvd
cp dvd/install.amd/initrd.gz temp/
cd temp/
gunzip initrd.gz
mv initrd initrd.img
mkdir initrd
cd initrd/
cpio -id < ../initrd.img
```


Debian
```bash
wget http://mirrors.163.com//debian/dists/jessie/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz
wget http://mirrors.163.com//debian/dists/jessie/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux
#cp amd64/debian/debian/install.amd/initrd.gz .
#cp amd64/debian/debian/install.amd/vmlinuz .
gunzip initrd.gz
mv initrd initrd.img
mkdir initrhttp://mirrors.163.com/d
cd initrd/
cpio -id < ../initrd.img

## copy pressed.cfg
cp ../preseed.cfg .

vim bin/fetch-url

## 从DVD中拷贝hdd的驱动
cp -rn ../temp/initrd/lib/modules/3.16.0-4-amd64/kernel/* lib/modules/3.16.0-4-amd64/kernel/

## 打包
find . | cpio -o -H newc | gzip -9 > ../initrd.gz
```


vim bin/fetch-url
```
url="$1"
dst="$2"

## add 2 lines
prefix="/cdrom"
url=${url#$prefix}
```

preseed.cfg
```
#locate 
d-i debian-installer/locale string en_US
d-i debian-installer/language string en
d-i debian-installer/country string CN

#keyboard 
d-i console-setup/ask_detect boolean false 
d-i console-configuration/layoutcode string us 
d-i console-keymaps-at/keymap select us
d-i keyboard-configuration/xkb-keymap select us

#clock 
d-i clock-setup/utc boolean false 
d-i time/zone string Asia/Shanghai 

#network 
d-i netcfg/choose_interface select eth0 
d-i netcfg/dhcp_failed note 
d-i netcfg/dhcp_options select Configure network manually 
#d-i netcfg/get_hostname string db86
#d-i netcfg/get_domain string pxe_debian
d-i netcfg/hostname string db86



#mirror 
d-i mirror/country string manual 
d-i mirror/http/hostname string 192.168.1.20
d-i mirror/http/directory string /debian/8.6/amd64 
#d-i mirror/http/hostname string example.com
#d-i mirror/http/directory string /pub/mirrors/ubuntu
d-i mirror/http/proxy string

#clock 
d-i clock-setup/ntp boolean true 

#Partitioning 
#d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-auto/choose_recipe select atomic
d-i partman/default_filesystem string ext4
d-i partman/confirm_write_new_label boolean true
d-i partman/choose_partition select Finish 
#d-i partman/confirm boolean true 
d-i partman/confirm_nooverwrite boolean true 

# Base system installation 
d-i base-installer/kernel/image string linux-generic 

#user 
d-i passwd/root-login boolean root 
d-i passwd/root-password password pass123
d-i passwd/root-password-again password pass123 
d-i user-setup/allow-password-weak boolean true 
d-i passwd/make-user boolean true
d-i user-setup/encrypt-home boolean false 

# create user
d-i passwd/user-fullname string media User
d-i passwd/username string media 

# common user
d-i passwd/user-password password pass123
d-i passwd/user-password-again password pass123


#file system 
#d-i live-installer/net-image string http://192.168.1.20/ubuntu14.04/install/filesystem.squashfs 

#apt setup 
d-i apt-setup/contrib boolean true
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true
d-i apt-setup/backports boolean true

#d-i apt-setup/use_mirror boolean false 
#d-i apt-setup/services-select multiselect security
#d-i apt-setup/security_host string example.com/pub/mirrors
#d-i apt-setup/security_path string /ubuntu

d-i debian-installer/allow_unauthenticated string true 

#package 
tasksel tasksel/first multiselect standard 
d-i pkgsel/include string openssh-server vim 
d-i pkgsel/install-language-support boolean false 
d-i pkgsel/language-packs multiselect en, zh 
d-i pkgsel/update-policy select none 

#grub 
d-i grub-installer/skip boolean false 
d-i lilo-installer/skip boolean true 
d-i grub-installer/grub2_instead_of_grup_legacy boolean true 
d-i grub-installer/only_debian boolean true 
d-i grub-installer/with_other_os boolean true 
d-i grub-installer/bootdev  string /dev/sda

### Running custom commands.
d-i preseed/late_command string in-target mkdir -p /root/.ssh; \
in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3NzaC1IAg1wilR9asDXIPwTsvZXasdTXqasdKv0rIqqweAtxGVgup foobar' >> /root/.ssh/authorized_keys"; \
in-target chown -R root:root /root/.ssh/


# custom
#d-i preseed/late_command string echo "hello" >> /target/root/test
#d-i preseed/late_command string echo 'Acquire::http::proxy::example.com "DIRECT";' >> /target/etc/apt/apt.conf 

d-i finish-install/reboot_in_progress note 
```


CentOS6.5 kickstart file
```
# Kickstart file automatically generated by anaconda.

#version=DEVEL
install
reboot
nfs --server=192.168.1.20 --dir=/var/lib/tftpboot/CentOS/6.5/x86_64
lang en_US.UTF-8
keyboard us
network --onboot yes --device eth0 --bootproto dhcp --noipv6
rootpw  --iscrypted $6$6hnmkS6Oqv8n2bnD$pBfBL3XoZW/wXJGHdOtVlFd61xrzK7QJ0SgoC0jeCYJupp1.9o.RsGC34svfvrssiMiLnXoZP.wQ3jt2qwaFw0
firewall --service=ssh
authconfig --enableshadow --passalgo=sha512
selinux --disabled
timezone --utc Asia/Shanghai
user --name=media --password=$6$UfwmGiXeGli2r3v0$E/3eldZ9Jkt5UrSN42wWIWBymcho5kWS..73T.B8EEB1rrU0CC56YETjt8aAd7EDIyk1nq8MHuqPBtaBUSDfL1 --iscrypted --gecos="media"
xconfig  --startxonboot

ignoredisk --only-use=sda
# Partition clearing information
clearpart --all --initlabel --drives=sda
part /boot --fstype ext4 --size 500
part swap --recommended
part / --fstype ext4 --size 120000 --grow


bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"
# The following is the partition information you requested
# Note that any partitions you deleted are not expressed
# here so unless you clear all partitions first, this is
# not guaranteed to work
#clearpart --all --drives=sda

#part /boot --fstype=ext4 --size=500
#part pv.008002 --grow --size=1
#volgroup vg_mediapxetest --pesize=4096 pv.008002
#logvol /home --fstype=ext4 --name=lv_home --vgname=vg_mediapxetest --grow --size=100
#logvol / --fstype=ext4 --name=lv_root --vgname=vg_mediapxetest --grow --size=1024 --maxsize=51200
#logvol swap --name=lv_swap --vgname=vg_mediapxetest --grow --size=7760 --maxsize=7760


repo --name="CentOS"  --baseurl=nfs:192.168.1.20:/var/lib/tftpboot/CentOS/6.5/x86_64 --cost=100

%packages
@base
@core
@debugging
@basic-desktop
@desktop-debugging
@desktop-platform
@directory-client
@fonts
@general-desktop
@graphical-admin-tools
@input-methods
@internet-applications
@internet-browser
@java-platform
@legacy-x
@network-file-system-client
@office-suite
@print-client
@remote-desktop-clients
@server-platform
@server-policy
@workstation-policy
@x11
mtools
pax
oddjob
wodim
sgpio
genisoimage
device-mapper-persistent-data
abrt-gui
samba-winbind
certmonger
pam_krb5
krb5-workstation
libXmu
%end
```






CentOS7.2 kickstart file
```
# Kickstart file automatically generated by anaconda.

#version=DEVEL
install
reboot
nfs --server=192.168.1.20 --dir=/var/lib/tftpboot/CentOS/7.2/x86_64
lang en_US.UTF-8
keyboard us
network --onboot yes --device eth0 --bootproto dhcp --noipv6
rootpw  --iscrypted $6$6hnmkS6Oqv8n2bnD$pBfBL3XoZW/wXJGHdOtVlFd61xrzK7QJ0SgoC0jeCYJupp1.9o.RsGC34svfvrssiMiLnXoZP.wQ3jt2qwaFw0
firewall --service=ssh
authconfig --enableshadow --passalgo=sha512
selinux --disabled
timezone --utc Asia/Shanghai
user --name=media --password=$6$UfwmGiXeGli2r3v0$E/3eldZ9Jkt5UrSN42wWIWBymcho5kWS..73T.B8EEB1rrU0CC56YETjt8aAd7EDIyk1nq8MHuqPBtaBUSDfL1 --iscrypted --gecos="media"
xconfig  --startxonboot

ignoredisk --only-use=sda
# Partition clearing information
clearpart --all --initlabel --drives=sda
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=500
part pv.49 --fstype="lvmpv" --ondisk=sda --size=1 --grow
volgroup centos --pesize=4096 pv.49
logvol /  --fstype="xfs" --grow --maxsize=51200 --size=1024 --name=root --vgname=centos
logvol swap  --fstype="swap" --size=7824 --name=swap --vgname=centos
logvol /home  --fstype="xfs" --grow --size=500 --name=home --vgname=centos

bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"
# The following is the partition information you requested
# Note that any partitions you deleted are not expressed
# here so unless you clear all partitions first, this is
# not guaranteed to work
#clearpart --all --drives=sda

#part /boot --fstype=ext4 --size=500
#part pv.008002 --grow --size=1
#volgroup vg_mediapxetest --pesize=4096 pv.008002
#logvol /home --fstype=ext4 --name=lv_home --vgname=vg_mediapxetest --grow --size=100
#logvol / --fstype=ext4 --name=lv_root --vgname=vg_mediapxetest --grow --size=1024 --maxsize=51200
#logvol swap --name=lv_swap --vgname=vg_mediapxetest --grow --size=7760 --maxsize=7760


repo --name="CentOS"  --baseurl=nfs:192.168.1.20:/var/lib/tftpboot/CentOS/7.2/x86_64 --cost=100

%packages
@^gnome-desktop-environment
@base
@compat-libraries
@core
@desktop-debugging
@development
@dial-up
@directory-client
@fonts
@gnome-desktop
@guest-agents
@guest-desktop-agents
@input-methods
@internet-browser
@java-platform
@multimedia
@network-file-system-client
@networkmanager-submodules
@print-client
@security-tools
@x11
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
```

Ubuntu16.04 pressed file
```
#locate 
d-i debian-installer/locale string en_US 
#keyboard 
d-i console-setup/ask_detect boolean false 
d-i console-configuration/layoutcode string us 
d-i keyboard-configuration/modelcode string SKIP 
#clock 
d-i clock-setup/utc boolean false 
d-i time/zone string Asia/Shanghai 
#network 
d-i netcfg/choose_interface select eth0 
d-i netcfg/dhcp_failed note 
d-i netcfg/dhcp_options select Configure network manually 
d-i netcfg/get_hostname string ub16


#mirror 
d-i mirror/country string manual 
d-i mirror/http/hostname string 192.168.1.20
d-i mirror/http/directory string /ubuntu/16.04/amd64 
#d-i mirror/http/hostname string example.com
#d-i mirror/http/directory string /pub/mirrors/ubuntu
d-i mirror/http/proxy string

#clock 
d-i clock-setup/ntp boolean true 
#Partitioning 
#d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-auto/choose_recipe select atomic
d-i partman/default_filesystem string ext4
d-i partman/confirm_write_new_label boolean true
d-i partman/choose_partition select Finish 
d-i partman/confirm boolean true 
d-i partman/confirm_nooverwrite boolean true 
# Base system installation 
d-i base-installer/kernel/image string linux-generic 

#user 
d-i passwd/root-login boolean root 
d-i passwd/root-password password pass123
d-i passwd/root-password-again password pass123 
d-i user-setup/allow-password-weak boolean true 
d-i passwd/make-user boolean true
d-i user-setup/encrypt-home boolean false 

# create user
d-i passwd/user-fullname string media User
d-i passwd/username string media 

# common user
d-i passwd/user-password password pass123
d-i passwd/user-password-again password pass123


#file system 
#d-i live-installer/net-image string http://192.168.1.20/ubuntu14.04/install/filesystem.squashfs 

#apt setup 
d-i apt-setup/contrib boolean true
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true
d-i apt-setup/backports boolean true

#d-i apt-setup/use_mirror boolean false 
#d-i apt-setup/services-select multiselect security
#d-i apt-setup/security_host string example.com/pub/mirrors
#d-i apt-setup/security_path string /ubuntu

d-i debian-installer/allow_unauthenticated string true 

#package 
tasksel tasksel/first multiselect standard 
d-i pkgsel/include string openssh-server vim 
d-i pkgsel/install-language-support boolean false 
d-i pkgsel/language-packs multiselect en, zh 
d-i pkgsel/update-policy select none 

#grub 
d-i grub-installer/skip boolean false 
d-i lilo-installer/skip boolean true 
d-i grub-installer/grub2_instead_of_grup_legacy boolean true 
d-i grub-installer/only_debian boolean true 
d-i grub-installer/with_other_os boolean true 

### Running custom commands.
d-i preseed/late_command string in-target mkdir -p /root/.ssh; \
in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3NzaC1IAg1wilR9asDXIPwTsvZXasdTXqasdKv0rIqqweAtxGVgup foobar' >> /root/.ssh/authorized_keys"; \
in-target chown -R root:root /root/.ssh/


# custom
#d-i preseed/late_command string echo "hello" >> /target/root/test
#d-i preseed/late_command string echo 'Acquire::http::proxy::example.com "DIRECT";' >> /target/etc/apt/apt.conf 

d-i finish-install/reboot_in_progress note 
```


SUSE12 auto
```
<?xml version="1.0"?>
<!DOCTYPE profile>
<profile xmlns="http://www.suse.com/1.0/yast2ns" xmlns:config="http://www.suse.com/1.0/configns">
  <add-on>
    <add_on_products config:type="list"/>
  </add-on>
  <bootloader>
    <device_map config:type="list">
      <device_map_entry>
        <firmware>hd0</firmware>
        <linux>/dev/sda</linux>
      </device_map_entry>
      <device_map_entry>
        <firmware>hd1</firmware>
        <linux>/dev/sdb</linux>
      </device_map_entry>
    </device_map>
    <global>
      <activate>true</activate>
      <append>   resume=/dev/sda3 splash=silent quiet showopts</append>
      <append_failsafe>showopts apm=off noresume edd=off powersaved=off nohz=off highres=off processor.max_cstate=1 nomodeset x11failsafe</append_failsafe>
      <boot_boot>false</boot_boot>
      <boot_extended>false</boot_extended>
      <boot_mbr>true</boot_mbr>
      <boot_root>false</boot_root>
      <default>0</default>
      <distributor>SLES12</distributor>
      <generic_mbr>false</generic_mbr>
      <gfxmode>auto</gfxmode>
      <os_prober>false</os_prober>
      <terminal>gfxterm</terminal>
      <timeout config:type="integer">8</timeout>
      <vgamode/>
    </global>
    <loader_type>grub2</loader_type>
    <sections config:type="list"/>
  </bootloader>
  <deploy_image>
    <image_installation config:type="boolean">false</image_installation>
  </deploy_image>
  <firewall>
    <FW_ALLOW_FW_BROADCAST_DMZ>no</FW_ALLOW_FW_BROADCAST_DMZ>
    <FW_ALLOW_FW_BROADCAST_EXT>no</FW_ALLOW_FW_BROADCAST_EXT>
    <FW_ALLOW_FW_BROADCAST_INT>no</FW_ALLOW_FW_BROADCAST_INT>
    <FW_CONFIGURATIONS_DMZ/>
    <FW_CONFIGURATIONS_EXT/>
    <FW_CONFIGURATIONS_INT/>
    <FW_DEV_DMZ/>
    <FW_DEV_EXT/>
    <FW_DEV_INT/>
    <FW_FORWARD_ALWAYS_INOUT_DEV/>
    <FW_FORWARD_MASQ/>
    <FW_IGNORE_FW_BROADCAST_DMZ>no</FW_IGNORE_FW_BROADCAST_DMZ>
    <FW_IGNORE_FW_BROADCAST_EXT>yes</FW_IGNORE_FW_BROADCAST_EXT>
    <FW_IGNORE_FW_BROADCAST_INT>no</FW_IGNORE_FW_BROADCAST_INT>
    <FW_IPSEC_TRUST>no</FW_IPSEC_TRUST>
    <FW_LOAD_MODULES/>
    <FW_LOG_ACCEPT_ALL>no</FW_LOG_ACCEPT_ALL>
    <FW_LOG_ACCEPT_CRIT>yes</FW_LOG_ACCEPT_CRIT>
    <FW_LOG_DROP_ALL>no</FW_LOG_DROP_ALL>
    <FW_LOG_DROP_CRIT>yes</FW_LOG_DROP_CRIT>
    <FW_MASQUERADE>no</FW_MASQUERADE>
    <FW_PROTECT_FROM_INT>no</FW_PROTECT_FROM_INT>
    <FW_ROUTE>no</FW_ROUTE>
    <FW_SERVICES_ACCEPT_DMZ/>
    <FW_SERVICES_ACCEPT_EXT/>
    <FW_SERVICES_ACCEPT_INT/>
    <FW_SERVICES_ACCEPT_RELATED_DMZ/>
    <FW_SERVICES_ACCEPT_RELATED_EXT/>
    <FW_SERVICES_ACCEPT_RELATED_INT/>
    <FW_SERVICES_DMZ_IP/>
    <FW_SERVICES_DMZ_RPC/>
    <FW_SERVICES_DMZ_TCP/>
    <FW_SERVICES_DMZ_UDP/>
    <FW_SERVICES_EXT_IP/>
    <FW_SERVICES_EXT_RPC/>
    <FW_SERVICES_EXT_TCP/>
    <FW_SERVICES_EXT_UDP/>
    <FW_SERVICES_INT_IP/>
    <FW_SERVICES_INT_RPC/>
    <FW_SERVICES_INT_TCP/>
    <FW_SERVICES_INT_UDP/>
    <enable_firewall config:type="boolean">false</enable_firewall>
    <start_firewall config:type="boolean">false</start_firewall>
  </firewall>
  <general>
    <ask-list config:type="list"/>
    <mode>
      <confirm config:type="boolean">false</confirm>
    </mode>
    <proposals config:type="list"/>
    <signature-handling>
      <accept_file_without_checksum config:type="boolean">true</accept_file_without_checksum>
      <accept_non_trusted_gpg_key config:type="boolean">true</accept_non_trusted_gpg_key>
      <accept_unknown_gpg_key config:type="boolean">true</accept_unknown_gpg_key>
      <accept_unsigned_file config:type="boolean">true</accept_unsigned_file>
      <accept_verification_failed config:type="boolean">false</accept_verification_failed>
      <import_gpg_key config:type="boolean">true</import_gpg_key>
    </signature-handling>
    <storage>
      <partition_alignment config:type="symbol">align_optimal</partition_alignment>
      <start_multipath config:type="boolean">false</start_multipath>
    </storage>
  </general>
  <groups config:type="list">
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>100</gid>
      <group_password>x</group_password>
      <groupname>users</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>495</gid>
      <group_password>x</group_password>
      <groupname>nscd</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>21</gid>
      <group_password>x</group_password>
      <groupname>console</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>15</gid>
      <group_password>x</group_password>
      <groupname>shadow</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>62</gid>
      <group_password>x</group_password>
      <groupname>man</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>10</gid>
      <group_password>x</group_password>
      <groupname>wheel</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>497</gid>
      <group_password>x</group_password>
      <groupname>tape</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>498</gid>
      <group_password>x</group_password>
      <groupname>sshd</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>17</gid>
      <group_password>x</group_password>
      <groupname>audio</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>5</gid>
      <group_password>x</group_password>
      <groupname>tty</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>65534</gid>
      <group_password>x</group_password>
      <groupname>nogroup</groupname>
      <userlist>nobody</userlist>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>13</gid>
      <group_password>x</group_password>
      <groupname>news</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>65533</gid>
      <group_password>x</group_password>
      <groupname>nobody</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>33</gid>
      <group_password>x</group_password>
      <groupname>video</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>22</gid>
      <group_password>x</group_password>
      <groupname>utmp</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>0</gid>
      <group_password>x</group_password>
      <groupname>root</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>42</gid>
      <group_password>x</group_password>
      <groupname>trusted</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>9</gid>
      <group_password>x</group_password>
      <groupname>kmem</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>7</gid>
      <group_password>x</group_password>
      <groupname>lp</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>12</gid>
      <group_password>x</group_password>
      <groupname>mail</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>16</gid>
      <group_password>x</group_password>
      <groupname>dialout</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>19</gid>
      <group_password>x</group_password>
      <groupname>floppy</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>8</gid>
      <group_password>x</group_password>
      <groupname>www</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>1</gid>
      <group_password>x</group_password>
      <groupname>bin</groupname>
      <userlist>daemon</userlist>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>496</gid>
      <group_password>x</group_password>
      <groupname>polkitd</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>41</gid>
      <group_password>x</group_password>
      <groupname>xok</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>49</gid>
      <group_password>x</group_password>
      <groupname>ftp</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>3</gid>
      <group_password>x</group_password>
      <groupname>sys</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>499</gid>
      <group_password>x</group_password>
      <groupname>messagebus</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>32</gid>
      <group_password>x</group_password>
      <groupname>public</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>2</gid>
      <group_password>x</group_password>
      <groupname>daemon</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>14</gid>
      <group_password>x</group_password>
      <groupname>uucp</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>43</gid>
      <group_password>x</group_password>
      <groupname>modem</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>40</gid>
      <group_password>x</group_password>
      <groupname>games</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>6</gid>
      <group_password>x</group_password>
      <groupname>disk</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>20</gid>
      <group_password>x</group_password>
      <groupname>cdrom</groupname>
      <userlist/>
    </group>
    <group>
      <encrypted config:type="boolean">true</encrypted>
      <gid>54</gid>
      <group_password>x</group_password>
      <groupname>lock</groupname>
      <userlist/>
    </group>
  </groups>
  <kdump>
    <add_crash_kernel config:type="boolean">true</add_crash_kernel>
    <crash_kernel>224M-:112M</crash_kernel>
    <general>
      <KDUMP_COMMANDLINE/>
      <KDUMP_COMMANDLINE_APPEND/>
      <KDUMP_COPY_KERNEL>yes</KDUMP_COPY_KERNEL>
      <KDUMP_DUMPFORMAT>lzo</KDUMP_DUMPFORMAT>
      <KDUMP_DUMPLEVEL>31</KDUMP_DUMPLEVEL>
      <KDUMP_FREE_DISK_SIZE>64</KDUMP_FREE_DISK_SIZE>
      <KDUMP_IMMEDIATE_REBOOT>yes</KDUMP_IMMEDIATE_REBOOT>
      <KDUMP_KEEP_OLD_DUMPS>5</KDUMP_KEEP_OLD_DUMPS>
      <KDUMP_KERNELVER/>
      <KDUMP_NOTIFICATION_CC/>
      <KDUMP_NOTIFICATION_TO/>
      <KDUMP_SAVEDIR>file:///var/crash</KDUMP_SAVEDIR>
      <KDUMP_SMTP_PASSWORD/>
      <KDUMP_SMTP_SERVER/>
      <KDUMP_SMTP_USER/>
      <KDUMP_TRANSFER/>
      <KDUMP_VERBOSE>3</KDUMP_VERBOSE>
      <KEXEC_OPTIONS/>
    </general>
  </kdump>
  <keyboard>
    <keyboard_values>
      <delay/>
      <discaps config:type="boolean">false</discaps>
      <numlock>bios</numlock>
      <rate/>
    </keyboard_values>
    <keymap>english-us</keymap>
  </keyboard>
  <language>
    <language>en_US</language>
    <languages/>
  </language>
  <login_settings/>
  <networking>
    <dns>
      <dhcp_hostname config:type="boolean">false</dhcp_hostname>
      <resolv_conf_policy/>
      <write_hostname config:type="boolean">false</write_hostname>
    </dns>
    <interfaces config:type="list">
      <interface>
        <bootproto>dhcp</bootproto>
        <device>eth0</device>
        <dhclient_set_default_route>yes</dhclient_set_default_route>
        <startmode>auto</startmode>
      </interface>
      <interface>
        <bootproto>static</bootproto>
        <broadcast>127.255.255.255</broadcast>
        <device>lo</device>
        <firewall>no</firewall>
        <ipaddr>127.0.0.1</ipaddr>
        <netmask>255.0.0.0</netmask>
        <network>127.0.0.0</network>
        <prefixlen>8</prefixlen>
        <startmode>nfsroot</startmode>
        <usercontrol>no</usercontrol>
      </interface>
    </interfaces>
    <ipv6 config:type="boolean">true</ipv6>
    <keep_install_network config:type="boolean">false</keep_install_network>
    <managed config:type="boolean">false</managed>
    <net-udev config:type="list">
      <rule>
        <name>eth0</name>
        <rule>ATTR{address}</rule>
        <value>ac:22:0b:4d:59:3c</value>
      </rule>
    </net-udev>
    <routing>
      <ipv4_forward config:type="boolean">false</ipv4_forward>
      <ipv6_forward config:type="boolean">false</ipv6_forward>
    </routing>
  </networking>
  <ntp-client>
    <ntp_policy>auto</ntp_policy>
    <peers config:type="list"/>
    <start_at_boot config:type="boolean">true</start_at_boot>
    <start_in_chroot config:type="boolean">false</start_in_chroot>
    <sync_interval config:type="integer">5</sync_interval>
    <synchronize_time config:type="boolean">false</synchronize_time>
  </ntp-client>
  <partitioning config:type="list">
    <drive>
      <device>/dev/sda</device>
      <disklabel>msdos</disklabel>
      <enable_snapshots config:type="boolean">true</enable_snapshots>
      <initialize config:type="boolean">true</initialize>
      <partitions config:type="list">
        <partition>
          <create config:type="boolean">true</create>
          <crypt_fs config:type="boolean">false</crypt_fs>
          <filesystem config:type="symbol">xfs</filesystem>
          <format config:type="boolean">true</format>
          <loop_fs config:type="boolean">false</loop_fs>
          <mount>/</mount>
          <mountby config:type="symbol">uuid</mountby>
          <partition_id config:type="integer">131</partition_id>
          <partition_nr config:type="integer">1</partition_nr>
          <resize config:type="boolean">true</resize>
          <size>max</size>
        </partition>
        <partition>
          <create config:type="boolean">true</create>
          <crypt_fs config:type="boolean">false</crypt_fs>
          <filesystem config:type="symbol">xfs</filesystem>
          <format config:type="boolean">true</format>
          <loop_fs config:type="boolean">false</loop_fs>
          <mount>/boot</mount>
          <mountby config:type="symbol">uuid</mountby>
          <partition_id config:type="integer">131</partition_id>
          <partition_nr config:type="integer">2</partition_nr>
          <resize config:type="boolean">false</resize>
          <size>518159872</size>
        </partition>
        <partition>
          <create config:type="boolean">true</create>
          <crypt_fs config:type="boolean">false</crypt_fs>
          <filesystem config:type="symbol">swap</filesystem>
          <format config:type="boolean">true</format>
          <loop_fs config:type="boolean">false</loop_fs>
          <mount>swap</mount>
          <mountby config:type="symbol">uuid</mountby>
          <partition_id config:type="integer">130</partition_id>
          <partition_nr config:type="integer">3</partition_nr>
          <resize config:type="boolean">false</resize>
          <size>8578563584</size>
        </partition>
      </partitions>
      <pesize/>
      <type config:type="symbol">CT_DISK</type>
      <use>all</use>
    </drive>
  </partitioning>
  <proxy>
    <enabled config:type="boolean">false</enabled>
    <ftp_proxy/>
    <http_proxy/>
    <https_proxy/>
    <no_proxy>localhost, 127.0.0.1, example.com</no_proxy>
    <proxy_password/>
    <proxy_user/>
  </proxy>
  <report>
    <errors>
      <log config:type="boolean">true</log>
      <show config:type="boolean">true</show>
      <timeout config:type="integer">0</timeout>
    </errors>
    <messages>
      <log config:type="boolean">true</log>
      <show config:type="boolean">true</show>
      <timeout config:type="integer">10</timeout>
    </messages>
    <warnings>
      <log config:type="boolean">true</log>
      <show config:type="boolean">true</show>
      <timeout config:type="integer">10</timeout>
    </warnings>
    <yesno_messages>
      <log config:type="boolean">true</log>
      <show config:type="boolean">true</show>
      <timeout config:type="integer">10</timeout>
    </yesno_messages>
  </report>
  <services-manager>
    <default_target>graphical</default_target>
    <services>
      <disable config:type="list"/>
      <enable config:type="list">
        <service>sshd</service>
      </enable>
    </services>
  </services-manager>
  <software>
    <image/>
    <instsource/>
    <packages config:type="list">
      <package>irqbalance</package>
      <package>glibc</package>
      <package>openssh</package>
      <package>grub2</package>
      <package>syslinux</package>
      <package>kdump</package>
      <package>kexec-tools</package>
      <package>perl-Bootloader-YAML</package>
    </packages>
    <patterns config:type="list">
      <pattern>32bit</pattern>
      <pattern>Minimal</pattern>
      <pattern>apparmor</pattern>
      <pattern>documentation</pattern>
      <pattern>gnome-basic</pattern>
      <pattern>x11</pattern>
    </patterns>
  </software>
  <timezone>
    <hwclock>UTC</hwclock>
    <timezone>Asia/Shanghai</timezone>
  </timezone>
  <user_defaults>
    <expire/>
    <group>100</group>
    <groups/>
    <home>/home</home>
    <inactive>-1</inactive>
    <no_groups config:type="boolean">true</no_groups>
    <shell>/bin/bash</shell>
    <skel>/etc/skel</skel>
    <umask>022</umask>
  </user_defaults>
  <users config:type="list">
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>media</fullname>
      <gid>100</gid>
      <home>/home/media</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact>-1</inact>
        <max>99999</max>
        <min>0</min>
        <warn>7</warn>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>1000</uid>
      <user_password>$6$N8Y7HqDi13V2$QxUj72wHZB2pJeBWmZytE2UN6I0WFrZZ5FcYlbDqO4mhCmh95OF.vGQOsxiBwpEfGBiHkuy7mrtvta9zDpEtT.</user_password>
      <username>media</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>root</fullname>
      <gid>0</gid>
      <home>/root</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>0</uid>
      <user_password>$6$zbC5mUt4gwNV$DQh5Mw2rz1h3COn2ttiHx3hrzLWWS.gB/YmF12ZnF/ODKizl1RiFORvJsQZrXLxPQstBvQ9BCrsqpBKtXQ1Ue/</user_password>
      <username>root</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>WWW daemon apache</fullname>
      <gid>8</gid>
      <home>/var/lib/wwwrun</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/false</shell>
      <uid>30</uid>
      <user_password>*</user_password>
      <username>wwwrun</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>User for D-Bus</fullname>
      <gid>499</gid>
      <home>/var/run/dbus</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/false</shell>
      <uid>499</uid>
      <user_password>!</user_password>
      <username>messagebus</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Games account</fullname>
      <gid>100</gid>
      <home>/var/games</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>12</uid>
      <user_password>*</user_password>
      <username>games</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>nobody</fullname>
      <gid>65533</gid>
      <home>/var/lib/nobody</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>65534</uid>
      <user_password>*</user_password>
      <username>nobody</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Daemon</fullname>
      <gid>2</gid>
      <home>/sbin</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>2</uid>
      <user_password>*</user_password>
      <username>daemon</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Unix-to-Unix CoPy system</fullname>
      <gid>14</gid>
      <home>/etc/uucp</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>10</uid>
      <user_password>*</user_password>
      <username>uucp</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>News system</fullname>
      <gid>13</gid>
      <home>/etc/news</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>9</uid>
      <user_password>*</user_password>
      <username>news</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>User for polkitd</fullname>
      <gid>496</gid>
      <home>/var/lib/polkit</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/sbin/nologin</shell>
      <uid>497</uid>
      <user_password>!</user_password>
      <username>polkitd</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Manual pages viewer</fullname>
      <gid>62</gid>
      <home>/var/cache/man</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>13</uid>
      <user_password>*</user_password>
      <username>man</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>bin</fullname>
      <gid>1</gid>
      <home>/bin</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>1</uid>
      <user_password>*</user_password>
      <username>bin</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>SSH daemon</fullname>
      <gid>498</gid>
      <home>/var/lib/sshd</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/false</shell>
      <uid>498</uid>
      <user_password>!</user_password>
      <username>sshd</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>FTP account</fullname>
      <gid>49</gid>
      <home>/srv/ftp</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>40</uid>
      <user_password>*</user_password>
      <username>ftp</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>User for nscd</fullname>
      <gid>495</gid>
      <home>/run/nscd</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/sbin/nologin</shell>
      <uid>496</uid>
      <user_password>!</user_password>
      <username>nscd</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Printing daemon</fullname>
      <gid>7</gid>
      <home>/var/spool/lpd</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/bash</shell>
      <uid>4</uid>
      <user_password>*</user_password>
      <username>lp</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>Mailer daemon</fullname>
      <gid>12</gid>
      <home>/var/spool/clientmqueue</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/bin/false</shell>
      <uid>8</uid>
      <user_password>*</user_password>
      <username>mail</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>openslp daemon</fullname>
      <gid>2</gid>
      <home>/var/lib/empty</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/sbin/nologin</shell>
      <uid>494</uid>
      <user_password>!</user_password>
      <username>openslp</username>
    </user>
    <user>
      <encrypted config:type="boolean">true</encrypted>
      <fullname>user for rpcbind</fullname>
      <gid>65534</gid>
      <home>/var/lib/empty</home>
      <password_settings>
        <expire/>
        <flag/>
        <inact/>
        <max/>
        <min/>
        <warn/>
      </password_settings>
      <shell>/sbin/nologin</shell>
      <uid>495</uid>
      <user_password>!</user_password>
      <username>rpc</username>
    </user>
  </users>
</profile>
```






