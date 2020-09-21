# EFI

[TOC]

## 引进两个概念[^1]
### 计算机默认引导 
就是不管你的计算机有没有操作系统，定义了UEFI启动后将通过Bootx64.efi 引导你的计算机，并进入各种模式，维护、安装、计算机或者系统
这里是 Bootx64.efi ，它只是一个通用名，**权限丰富且大于Windows 默认**，就是说如果你的Windows 默认的启动文件不在了， 启动计算机默认的引导文件Bootx64.efi 也是可以启动计算机的。使用计算机默认文件随时可以在各种环境下启动计算机，EFI SHELL、ISO、 Windows、Linux... 都可以，通吃型.

### Windows默认引导
就是你为计算机安装了操作系统，或者修复了UEFI引导后，启动菜单会有 Windows Boot Manager 选项，该选项默认从bootmgfw.efi 启动系统 bootmgfw.efi  该位置的该文件只能用于启动Windows，**不是通用名，权限单一**

### Linux 默认引导
ubuntu系统 默认从 shimx64.efi 启动的
```sh
root@example.com:/boot/efi/EFI# tree
.
├── BOOT
│   ├── BOOTX64.EFI
│   ├── fbx64.efi
│   └── mmx64.efi
└── ubuntu
    ├── BOOTX64.CSV
    ├── grub.cfg
    ├── grubx64.efi
    ├── mmx64.efi
    └── shimx64.efi
```

按照设置里的顺序，找启动项。启动项分两种，设备启动项和文件启动项[^2]：
### 文件启动项
大约记录的是某个磁盘的某个分区的某个路径下的某个文件。对于文件启动项，固件会直接加载这个EFI文件，并执行。类似于DOS下你敲了个win.com就执行了Windows 3.2/95/98的启动。文件不存在则失败。

### 设备启动项
大约记录的就是“某个U盘”、“某个硬盘”。（此处只讨论U盘、硬盘）对于设备启动项，UEFI标准规定了默认的路径“\EFI\Boot\bootX64.efi”。UEFI会加载磁盘上的这个文件。文件不存在则失败。

## efi引导过程[^3]
```
                  +-----+
                  | 开机 |
                  +-----+
                     |
                     v
                  +------+
                  | UEFI |
                  +------+
                     |
                     v
                  硬件初始化
                     |
                     v
                  查找ESP
                     |
                     v
查找/EFI/XXX文件夹下的.efi文件(grub是grubx64.efi)
                     |
                     v
                  +------+
                  | grub |
                  +------+
                     |
                     v
      加载/boot/grub/的模块以及配置文件
                     |
                     v
           根据配置加载操作系统内核
                     +
                     v
                +----+----+
                | 操作系统 |
                +---------+
```

### Linux efibootmgr
```sh
root@example.com:/boot/efi/EFI# efibootmgr -v
BootCurrent: 0003
Timeout: 1 seconds
BootOrder: 0003,0004,0005,0006,0007
Boot0003* ubuntu        HD(1,GPT,6dad458c-37df-483e-ba67-2dd9263860d4,0x800,0x100000)/File(\EFI\UBUNTU\SHIMX64.EFI)
Boot0004* UEFI: PXE IP4 Intel(R) Ethernet Server Adapter I350-T4 NIC1   PciRoot(0x3)/Pci(0x0,0x0)/Pci(0x0,0x0)/MAC(b496913c2ba4,0)/IPv4(0.0.0.00.0.0.0,0,0)..BO
Boot0005* UEFI: PXE IP4 Intel(R) Ethernet Server Adapter I350-T4 NIC2   PciRoot(0x3)/Pci(0x0,0x0)/Pci(0x0,0x1)/MAC(b496913c2ba5,0)/IPv4(0.0.0.00.0.0.0,0,0)..BO
Boot0006* UEFI: PXE IP4 Intel(R) Ethernet Server Adapter I350-T4 NIC3   PciRoot(0x3)/Pci(0x0,0x0)/Pci(0x0,0x2)/MAC(b496913c2ba6,0)/IPv4(0.0.0.00.0.0.0,0,0)..BO
Boot0007* UEFI: PXE IP4 Intel(R) Ethernet Server Adapter I350-T4 NIC4   PciRoot(0x3)/Pci(0x0,0x0)/Pci(0x0,0x3)/MAC(b496913c2ba7,0)/IPv4(0.0.0.00.0.0.0,0,0)..BO
MirroredPercentageAbove4G: 0.00
MirrorMemoryBelow4GB: false
```



[^1]: 引用 http://bbs.wuyou.net/forum.php?mod=viewthread&tid=303679 作者: 2011hiboy
[^2]: 引用 https://zhuanlan.zhihu.com/p/31365115 作者: 王诗峣
[^3]: 引用 https://staight.github.io/2018/09/05/%E5%BC%95%E5%AF%BC%E6%80%BB%E7%BB%93/ 作者：staight


