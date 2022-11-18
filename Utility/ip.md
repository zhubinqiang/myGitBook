# ip 命令

[TOC]

```sh
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
                   netns | l2tp | macsec | tcp_metrics | token }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec |
                    -f[amily] { inet | inet6 | ipx | dnet | bridge | link } |
                    -4 | -6 | -I | -D | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -a[ll] }
```

这里的object主要有:

1. link ：关于装置 (device) 的相关设定，包括 MTU, MAC 地址等等
2. addr/address ：关于额外的 IP 协议，例如多 IP 的达成等等；
3. route ：与路由有关的相关设定

## ip link
ip link 可以设定与装置 (device) 有关的相关参数，包括 MTU 以及该网络接口的 MAC 等等，当然也可以启动 (up) 或关闭 (down) 某个网络接口。

```sh
[root@www ~]# ip [-s] link show  #单纯的查阅该装置相关的信息
[root@www ~]# ip link set [device] [动作与参数]
```

选项与参数：
- show：仅显示出这个装置的相关内容，如果加上 -s 会显示更多统计数据；
- set ：可以开始设定项目， device 指的是 eth0, eth1 等等界面代号；

动作与参数：包括有底下的这些动作：
+ up/down ：启动 (up) 或关闭 (down) 某个接口，其他参数使用默认的以太网络；
+ address ：如果这个装置可以更改 MAC 的话，用这个参数修改！
+ name ：给予这个装置一个特殊的名字；
+ mtu ：就是最大传输单元啊！


```sh
## 关闭网卡
ip link set enp0s25 down

## 换网卡名字
ip link set enp1s0 name eth0

## 开启网卡
ip link set eth up

## 设置 mac 地址
ip link set eth0 address aa:aa:aa:aa:aa:aa
```

## ip addr
```sh
[root@www ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp1s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:a0:a5:aa:47:2d brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.123/23 brd 10.67.117.255 scope global noprefixroute enp1s0f0
       valid_lft forever preferred_lft forever
    inet6 fe80::9e31:11d2:59da:3553/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
相关参数：主要有底下这些[^ip_addr]：
`<BROADCAST,MULTICAST,UP,LOWER_UP>` ： `BROADCAST` 表示该接口支持广播； `MULTICAST` 表示该接口支持多播； `UP` 表示该网络接口已启用； `LOWER_UP` 表示网络电缆已插入，设备已连接至网络
`mtu 1500`: 最大传输单位（数据包大小）为 1,500 字节
`qdisc pfifo_fast`：用于数据包排队
`state UP` ：网络接口已启用


broadcast：设定广播地址，如果设定值是 + 表示『让系统自动计算』
label ：亦即是这个装置的别名，例如 eth0:0 就是了！
scope ：这个界面的领域，通常是这几个大类：
global ：允许来自所有来源的联机；
site ：仅支持 IPv6 ，仅允许本主机的联机；
link ：仅允许本装置自我联机；
host ：仅允许本主机内部的联机；
所以当然是使用 global 啰！预设也是 global 啦！

与第三层网络层有关的参数啦！ 主要是在设定与 IP 有关的各项参数，包括 netmask, broadcast 等等。

```sh
[root@www ~]# ip addr show ## 就是查阅 IP 参数啊！
[root@www ~]# ip addr [add/del] [IP参数] [dev 装置名] [相关参数]
```

选项与参数：
- show ：单纯的显示出接口的 IP 信息啊；
- add/del ：进行相关参数的增加 (add) 或删除 (del) 设定，主要有：

IP 参数：主要就是网域的设定，例如 192.168.100.100/24 之类的设定喔；
dev ：这个 IP 参数所要设定的接口，例如 eth0, eth1 等等；

```sh
## 新增一个接口 名为 eth0:pxe
# 那个 broadcast + 也可以写成 broadcast 192.168.100.255
ip addr add 192.168.100.123/24 dev eth0 label eth0:pxe broadcast +

## 删除上面的接口
ip addr delete 192.168.100.123/24 dev eth0
```

##  ip route
路由的观察与设定！事实上 ip route 的功能几乎与 route 这个指令差不多，但是，他还可以进行额外的参数设计，例如 MTU 的规划等。

```sh
[root@www ~]# ip route show  ## 单纯的显示出路由的设定而已
[root@www ~]# ip route [add/del] [IP或网域] [via gateway] [dev装置]
```

选项与参数：
- show ：单纯的显示出路由表，也可以使用 list ；
- add/del ：增加 (add) 或删除 (del) 路由的意思。
- IP或网域：可使用 192.168.50.0/24 之类的网域或者是单纯的IP
- via ：从那个 gateway 出去，不一定需要
- dev ：由那个装置连出去，这就需要了
- mtu ：可以额外的设定 MTU 的数值

```sh
➜ ip route
default via 10.67.116.1 dev enp0s25 proto static metric 100
default via 192.168.100.100 dev eth0 proto static metric 101
10.67.116.0/23 dev enp0s25 proto kernel scope link src 10.67.116.23 metric 100
172.18.0.0/16 dev docker0 proto kernel scope link src 172.18.0.1
192.168.100.0/24 dev eth0 proto kernel scope link src 192.168.100.73 metric 100
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
```

- proto：此路由的路由协议，主要有 redirect, kernel, boot, static, ra 等， 其中 kernel 指的是直接由核心判断自动设定。
- scope：路由的范围，主要是 link ，亦即是与本装置有关的直接联机。
- metric: metric的值越小，优先级越高。如果两块网卡的Metric的值相同，就会出现抢占优先级继而网卡冲突，将会有一块网卡无法连接 。

```sh
## 添加局域网的路由
ip route add 192.168.100.0/24 dev eth0

## 添加通往外部的路由， 跨路由
ip route add 192.168.100.0/24 via 192.168.100.100 dev eth0

## 添加默认路由
ip route add default via 192.168.100.100 dev eth0

## 删除路由
ip route del 192.168.100.0/24
ip route del default via 192.168.100.100 dev eth0
```

> ip route 指令对路由的修改不能保存，重启就没了。

添加永久静态路由
```
[root@linux-node1 network-scripts]# cat /etc/sysconfig/network-scripts/route-eth0
10.18.196.0/255.255.254.0 via 192.168.56.11 dev eth0

[root@linux-node1 network-scripts]# nmcli dev connect eth0 # 重启计算机，或者重新启用设备 eth0 才能生效。

[root@linux-node1 network-scripts]# nmcli dev disconnect eth0 && nmcli dev connect eth0
# 一般直接连接一次设备即可，如果不成功就先断开设备再连接设备，注意必须两个指令一起运行
```


[^ip_addr]: https://www.codeplayer.org/Wiki/Router/Linux%20ip%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3.html
