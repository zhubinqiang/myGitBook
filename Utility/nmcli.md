# nmcli

[TOC]

CentOS7之前的网络管理是通过ifcfg文件配置管理接口(device)，而现在是通过NetworkManager服务管理连接(connection)。
一个接口(device)可以有多个连接(connection)，但是同时只允许一个连接(connection)处于激活（active）状态。

`nmtui` 这个命令是带有图形界面的。

## device
```sh
sudo nmcli device disconnect eth1
sudo nmcli device connect eth1
sudo nmcli dev list eth1
```

> **注意**: 建议使用 `nmcli dev disconnect iface iface-name` 命令，而不是 `nmcli con down id id-string` 命令，因为连接断开可将该接口放到“手动”模式，这样做用户让 NetworkManager 启动某个连接前，或发生外部事件（比如载波变化、休眠或睡眠）前，不会启动任何自动连接。 [^1]

```sh
➜  WS nmcli dev
DEVICE       TYPE      STATE         CONNECTION
docker0      bridge    connected     docker0
virbr0       bridge    connected     virbr0
enp0s31f6    ethernet  connected     enp0s31f6
wlp8s0       wifi      disconnected  --
veth4b7e343  ethernet  unmanaged     --
veth5a295ed  ethernet  unmanaged     --
vethfc0be2a  ethernet  unmanaged     --
lo           loopback  unmanaged     --
virbr0-nic   tun       unmanaged     --

➜  WS nmcli dev show enp0s31f6
GENERAL.DEVICE:                         enp0s31f6
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         E0:D5:5E:66:02:5D
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     enp0s31f6
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/1
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         10.67.112.48/23
IP4.GATEWAY:                            10.67.112.1
IP4.DNS[1]:                             10.248.2.5
IP6.ADDRESS[1]:                         fe80::82bd:725c:ac4f:3a03/64
IP6.GATEWAY:                            --
```

## connection
一个连接(connection)就是/etc/sysconfig/network-scripts/目录下的一个配置文件，  
接口(device)是物理设备，一个物理设备可以拥有多个配置文件，  
但只能有一个配置文件属于使用(active)状态；配置文件的生成与使用状态均由NetworkManager控制。 参考于[这里](https://www.jianshu.com/p/5d5560e9e26a)

> 对于Ubuntu系统 nmcli con 配置保存在 /etc/NetworkManager/system-connections/

```sh
sudo nmcli con add type ethernet con-name dynamic-bi ifname eth1
sudo nmcli con down dynamic-bi
sudo nmcli con up dynamic-bi
sudo nmcli con show dynamic-bi

sudo nmcli con add type ethernet con-name static-bi ifname eth1
sudo nmcli con modify static-bi \
    ipv4.method static \
    ipv4.addresses 192.168.0.5/24 \
    ipv4.dns "10.248.2.5" \
    connection.autoconnect no
sudo nmcli con down static-bi
sudo nmcli con up static-bi

sudo nmcli connection delete dynamic-bi
```

```sh
➜  WS nmcli con
NAME       UUID                                  TYPE            DEVICE
docker0    e9742bba-8ab5-4045-8fb8-a216de47a9c4  bridge          docker0
enp0s31f6  38254c6f-abad-40b1-a84b-4d654030bca8  802-3-ethernet  enp0s31f6
virbr0     049a2dde-3994-44b6-a8b3-5bb6fd786f67  bridge          virbr0
```


[^1]: 转载于 https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/sec-Using_the_NetworkManager_Command_Line_Tool_nmcli


