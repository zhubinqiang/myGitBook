# linux kernel

[TOC]

参考bootlin的教程 [^bootlin]


## module 载入时的参数
kernel command line 可以在 `/sys/module/<module-name>/parameters/`
每一个文件对应一个参数， 文件内容对应参数值
/sys/module/hello/parameters/


/sys/module/i915/parameters/

```
modprobe.blacklist=ast,snd_hda_intel i915.force_probe=* modprobe.blacklist=ast,snd_hda_intel intel_pstate=disable intel_idle.max_cstate=1 i915.enable_guc=2 i915.enable_rc6=0
```

github 上面一个 module的[源代码](https://github.com/bootlin/training-materials/blob/master/code/hello-param/hello_param.c)

```sh
sudo insmod hello.ko howmany=5 whom=Tom
```

```sh
echo 2 > /sys/module/hello/parameters/howmany
echo Jerry > /sys/module/hello/parameters/whom
```

/etc/modprobe.conf 或者 任一文件在/etc/modprobe.d/
```
options hello howmany=5 whom=Tom
```

或者在kernel的命令行添加
```
hello.howmany=3 hello.whom=jerry
```


## 模块
```sh
$ cat /proc/modules  | grep hello
hello 16384 0 - Live 0x0000000000000000 (OE)

$ modinfo hello
filename:       /lib/modules/5.15.0-58-generic/updates/dkms/hello.ko
license:        Dual BSD/GPL
srcversion:     31FE72DA6A560C890FF9B3F
depends:        
retpoline:      Y
name:           hello
vermagic:       5.13.0-52-generic SMP mod_unload modversions 
sig_id:         PKCS#7
signer:         media-mc3 Secure Boot Module Signature key
sig_key:        6F:D7:C0:55:9A:8A:44:C3:9A:A3:77:42:98:FD:39:D7:37:6C:92:17
sig_hashalgo:   sha512
signature:      0F:22:EA:7C:B0:50:06:3D:62:59:71:71:8C:54:C8:22:57:6A:9E:68:
                03:E4:06:B4:EA:BF:62:B0:CF:B5:4F:C9:06:96:32:69:34:A9:6D:72:
                A2:D8:EB:74:6C:99:41:26:A1:EB:D0:68:5B:AE:96:E2:54:40:0F:D1:
                BB:3A:EC:92:B5:05:7F:0F:27:F9:8B:65:DD:0B:6F:F5:6A:FE:82:BE:
                1D:4E:6B:F5:C7:C3:B9:C1:3C:79:28:CF:1C:18:A8:63:49:C3:D1:83:
                EF:62:05:89:2A:A0:DF:54:3D:84:51:6B:F5:5B:60:EC:08:CA:78:8D:
                4D:B8:CE:18:FA:72:56:AD:D8:89:A4:AD:31:DB:29:2A:A8:BA:B3:75:
                45:24:9D:18:FC:66:74:A1:62:CD:5B:F3:33:91:CB:46:CD:59:04:A6:
                D8:F7:B3:25:8E:BB:A3:23:88:AE:47:C6:A1:E6:39:29:59:1A:34:04:
                A7:08:14:9C:8F:C9:42:8E:44:15:2F:61:31:CF:2D:15:53:21:30:E0:
                C1:D2:C7:2A:EF:46:A7:FF:46:C8:39:90:D7:BB:52:06:7A:3C:8F:21:
                0C:6C:49:A1:A5:8A:A6:6B:71:81:77:77:6F:52:80:D8:14:B1:91:69:
                C7:A2:0E:51:EC:5C:E4:1D:3D:C8:7A:F3:24:3E:39:26
```


Q&A:

在 makefile 中 加入下面的配置避免安装模块过程中的签名问题
```
## to avoid this issue
# "module verification failed: signature and/or required key missing - tainting kernel"
CONFIG_MODULE_SIG=n
CONFIG_MODULE_SIG_ALL=n
```

开机启动
```
# /usr/lib/modules/5.15.0-67-generic/kernel/drivers/char
sudo cp hello.ko /usr/lib/modules/$(uname -r)/kernel/drivers/char

sudo depmod -a

echo "hello" >> /etc/modules
```


[^bootlin]: https://bootlin.com/doc/training/linux-kernel/linux-kernel-slides.pdf



