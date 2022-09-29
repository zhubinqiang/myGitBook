# QEMU

[TOC]

Quick EMUlator


## 创建 qcow image
```sh
qemu-img create -f qcow2 centos-7.4.qcow 30G
qemu-img info centos-7.4.qcow
```

## 启动 qemu
```bash
HDD=centos-7.4.qcow
ISO=CentOS-7-x86_64-DVD-1708.iso

qemu-system-x86_64 \
-m 4096 -smp 4 -M pc \
-name centos -cpu host -hda ${HDD} \
-enable-kvm \
-machine kernel_irqchip=on \
-serial /dev/ttyS0 \
-boot d \
-cdrom ${ISO}
```

> 启动  `Ctrl + Alt + G` 退出界面

## 快照
```sh
qemu-img snapshot -l centos-7.4.qcow
qemu-img snapshot -c snapshot01 centos-7.4.qcow
qemu-img info centos-7.4.qcow
```

```sh
## create snapshot02
qemu-img snapshot -c snapshot02 centos-7.4.qcow

## apply snapshot01
qemu-img snapshot -a snapshot01 centos-7.4.qcow

## delete snapshot02
qemu-img snapshot -d snapshot02 centos-7.4.qcow
```





