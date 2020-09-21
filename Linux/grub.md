# grub
[TOC]

## grub
grub 大一统启动加载器(**Gr**and **U**nified **B**ootloader)[^efi_define]

检查 `ls /sys/firmware/efi` 这个目录是否存在[^efi_exist], 有的话就是efi模式下启动的。

## 配置
grub 载入的配置文件: /boot/grub/grub.cfg
生成grub.cfg 的配置文件在：/etc/default/grub

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### 取消子菜单
```
GRUB_DISABLE_SUBMENU=y
```

### 调用之前的启动条目
GRUB 能够记住你最近一次使用的启动项，并且在下次启动时将其作为默认项[^grub_setting]。

```
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

### 修改默认启动项
1. 修改菜单标题
```
# GRUB_DEFAULT='Ubuntu, with Linux 4.15.0-20-generic'
GRUB_DEFAULT='Advanced options for Ubuntu>Ubuntu, with Linux 4.15.0-20-generic'
```
此方法有待商榷

2. 使用数字编号
```
GRUB_DEFAULT="1>2"
```

GRUB 启动项序号从 0 开始计数，0 代表第一个启动项，也是上述选项的默认值，1 表示第二个启动项，以此类推。主菜单和子菜单项之间用 > 隔开。

### 隐藏 GRUB 界面
```
# GRUB_HIDDEN_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden
```

不用等 GRUB 倒计时, timeout style 有 `GRUB_TIMEOUT_STYLE=[menu|countdown|hidden]`


## FAQ
### error: environment block too small
1. 备份或者删除 grubenv
```sh
cd /boot/grub/
sudo mv grubenv grubenv
```

2. 重建 grubenv
```sh
cd /boot/grub/
sudo grub-editenv grubenv create
```


[^efi_define]: https://wiki.archlinux.org/index.php/GRUB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
[^efi_exist]: https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
[^grub_setting]: https://wiki.archlinux.org/index.php/GRUB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)/Tips_and_tricks_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%B0%86_UUID_%E5%92%8C%E5%9F%BA%E7%A1%80%E8%84%9A%E6%9C%AC%E7%BB%93%E5%90%88%E4%BD%BF%E7%94%A8

