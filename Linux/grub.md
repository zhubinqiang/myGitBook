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

## 转交控制权
磁盘与分区在 grub2 中的代号 [^grub-hd]
|     代号     |                   说明                    |
| :----------- | :---------------------------------------- |
| (hd0,1)      | 一般的默认语法，由 grub2 自动判断分区格式 |
| (hd0,msdos1) | 此磁盘的分区为传统的 MBR 模式             |
| (hd0,gpt1)   | 此磁盘的分区为 GPT 模式                   |


硬盘搜寻顺序
| 硬盘搜寻顺序 |         在 Grub2 当中的代号         |
| :---------- | :---------------------------------- |
| 第一块 (MBR) | (hd0) (hd0,msdos1) (hd0,msdos2) ... |
| 第二块 (GPT) | (hd1) (hd1,gpt1) (hd1,gpt2) ...     |
| 第三块       | (hd2) (hd2,1) (hd2,2) (hd2,3) ...   |


```
menuentry 'Go to MBR' --id 'mbr' {
    insmod chain
    set root=(hd0)
    chainloader +1
}

menuentry 'Go to Windows 7' --id 'win7' {
    insmod chain
    insmod ntfs
    set root=(hd0,msdos2)
    chainloader +1
}
```

> 在配置文件/etc/default/grub中设置 `GRUB_DEFAULT=win7` 再执行 `grub2-mkconfig` 通过 --id 里面的内容

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
[^grub-hd]: https://wizardforcel.gitbooks.io/vbird-linux-basic-4e/content/168.html

