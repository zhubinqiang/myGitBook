# DKMS

[TOC]

DKMS 全称 Dynamic Kernel Module Support。
**动态内核模块支持** [^introduce] 是用来生成Linux的内核模块的一个框架，其源代码一般不在Linux内核源代码树。 
当新的内核安装时，DKMS支持的内核设备驱动程序 到时会自动重建。 
DKMS可以用在两个方向：如果一个新的内核版本安装，自动编译所有的模块，或安装新的模块（驱动程序）在现有的系统版本上，而不需要任何的手动编译或预编译软件包需要。
例如，这使得新的显卡可以使用在旧的Linux系统上。


DKMS的使用流程可以用下图[^chart]简单表示:
```
                                                  uninstall
                                             ┌─────────────────────┐
                                             │                     │
                                             ▼                     │
┌───────────┐     ┌─────────────┐     ┌────────────┐         ┌─────┴──────┐
│  Not In   │ add │    Added    │build│   Build    │ install │ Installed  │
│   Tree    ├────►│    State    ├────►│   State    ├────────►│  State     │
│           │     │             │     │            │         │            │
└───────────┘     └─────────────┘     └────────────┘         └─────┬──────┘
      ▲                                                            │
      │                                                            │
      └────────────────────────────────────────────────────────────┘
                            remove
```


## 安装 dkms
```bash
sudo apt install -y dkms
```

## 准备
hell.c

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void)
{
    printk(KERN_ALERT "Hello, world\n");
    return 0;
}

static void hello_exit(void)
{

    printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

Makefile
```makefile
obj-m := hello.o
KVERSION:= $(shell uname -r)
CONFIG_MODULE_SIG=n

all:
    $(MAKE) -C /lib/modules/$(KVERSION)/build M=$(PWD) modules
clean:
    $(MAKE) -C /lib/modules/$(KVERSION)/build M=$(PWD) clean
```

DKMS要求 **目录必须以 `<module>-<module-version>` 的格式命名** 放到 /usr/src/ 下面

本例在 /usr/src/hello-1.0.0
```bash
$ pwd
/usr/src/hello-1.0.0
$ tree
.
├── dkms.conf
├── hello.c
└── Makefile
```

```bash
sudo dkms add -m hello -v 1.0.0

sudo dkms install -m hello -v 1.0.0

sudo dkms remove -m hello -v 1.0.0 --all
sudo dkms remove hello/1.0.0 -k `uname -r`
```

查询状态
```bash
# dkms add -m hello -v 1.0.0
$ dkms status
hello, 1.0.0: added

# dkms build -m hello -v 1.0.0
$ dkms status
hello, 1.0.0, 5.13.0-52-generic, x86_64: built

# dkms install -m hello -v 1.0.0
$ dkms status
hello, 1.0.0, 5.13.0-52-generic, x86_64: installed
```


Use the dkms remove function before trying to build again.
```sh
ls /var/lib/dkms/
sudo rm -rf /var/lib/dkms/intel-platform-vsec-dkms
dkms status
```


## 引用
[^chart]: https://www.cnblogs.com/wwang/archive/2011/06/21/2085571.html
[^introduce]: https://zh.m.wikipedia.org/zh-hans/%E5%8A%A8%E6%80%81%E5%86%85%E6%A0%B8%E6%A8%A1%E5%9D%97%E6%94%AF%E6%8C%81


