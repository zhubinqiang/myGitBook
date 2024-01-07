# linux kernel

[TOC]

参考bootlin的教程 [^bootlin]

一个简单的内核模块的例子[^hello.c]
```c
// SPDX-License-Identifier: GPL-2.0
/* hello.c */
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

static int __init hello_init(void)
{
	pr_alert("Good morrow to this fair assembly.\n");
	return 0;
}

static void __exit hello_exit(void)
{
	pr_alert("Alas, poor world, what treasure hast thou lost!\n");
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Greeting module");
MODULE_AUTHOR("William Shakespeare");
```

对应的Makefile文件
```Makefile
obj-m := hello.o

KDIR := /lib/modules/`uname -r`/build
PWD := $(shell pwd)

default:
	$(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules
```

> 对于 Makefile 参考 Documentation/kbuild/modules.rst


函数修饰符`__init`（前面加下划线的表示这是给内核使用的函数），本质上是个宏定义，在内核源代码中就有`#define __init xxxx`。
这个`__init`的作用就是将被他修饰的函数放入`.init.text`段中去（本来默认情况下函数是被放入.text段中）[^__init]。

include/linux/init.h
```c
#define __init      __section(.init.text) __cold  __latent_entropy __noinitretpoline
```



## module 载入时的参数
kernel command line 可以在 `/sys/module/<module-name>/parameters/`
每一个文件对应一个参数， 文件内容对应参数值
/sys/module/hello/parameters/


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

开机不想载入可以写在配置文件中，或者kernel cmdline
```sh
echo 'blacklist i915' > /etc/modprobe.d/blacklist-i915.conf
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

## 编译
```sh
make modules_install INSTALL_MOD_PATH=/path/to/modules
make headers_install INSTALL_HDR_PATH=/path/to/headers
make install INSTALL_PATH=/path/to/kernel
```

## 可发现的硬件: USB和PCI
有用的Linux命令
`lsusb`: 列出所有探测到的USB设备
`lspci`: 列出所有探测到的PCI设备

> 探测到设备并不意味着有kernel驱动与之关联。


## Kernel and Device Drivers
The device model is organized around three main data structures:
- The struct bus_type structure, which represents one type of bus (USB, PCI, I2C, etc.)
- The struct device_driver structure, which represents one driver capable of handling certain devices on a certain bus.
- The struct device structure, which represents one device connected to a bus

**下面简要说明各自的概念及它们之间的关系**：
bus_type：
bus_type代表计算机中的一类总线类型。总线是一种通信系统，允许连接到它的不同设备进行数据交换。每种类型的总线都有其特定的协议和行为方式。在内核中，bus_type结构体定义了一组特定类型的总线所共有的操作和特性。

device_driver：
device_driver是指具体的设备驱动程序，它定义了内核如何与特定硬件设备通信的方法。设备驱动程序包含了操作硬件所需的指令和功能，它通知内核如何发送指令给硬件，以及如何读取硬件发送回来的数据。

device：
device代表系统中的一个具体硬件设备。在内核设备模型中，每个device结构体实例都代表了一个被系统识别的硬件设备，包含了描述这个设备的信息，比如它在总线上的位置、它的属性等。


它们之间的关系：
总线与设备：设备通过总线与计算机其他部分通信。因此，每个设备都关联到一个特定的bus_type，这表明它通过何种方式与计算机交换数据。例如，USB设备会关联到USB总线类型，PCI设备会关联到PCI总线类型。

总线与设备驱动：设备驱动程序同样关联到一个特定的bus_type，因为不同类型的总线有不同的协议，设备驱动需要知道如何通过特定的总线与设备通信。

设备与设备驱动：每个device都需要一个device_driver来管理它。内核通过匹配设备的信息（如ID、总线类型等）来为设备寻找正确的驱动程序。当合适的设备驱动被加载到内核时，它会注册自己以表明可以管理某类设备。然后内核会把相应的device实例与匹配的device_driver关联起来。


总的来说，bus_type定义了总线的行为和通信协议，device是连接到总线上的硬件设备的抽象，而device_driver是内核用来与特定设备通信的代码。内核设备模型负责将这些部分协调起来，确保硬件设备能够正确地被识别、驱动和管理。

举个例子：
当操作系统（例如Linux）中的应用程序需要向一个设备发送一段字符时，该过程涉及到bus_type、device_driver和device的交互。以下是一个简化的例子，说明这三者在数据流中的作用：


**示例场景**
假设有一个应用程序需要向一个通过USB连接的打印机发送一段文本以打印。

步骤和数据流

应用程序发起请求：
应用程序通过系统调用或其他机制告诉操作系统它需要发送数据到打印机。

操作系统定位设备和驱动：
操作系统首先确定打印机的device实例。这个实例包含了打印机的相关信息，比如它是通过USB总线连接的。
然后，操作系统查找适合该打印机的device_driver，这个驱动程序知道如何与USB总线上的打印机设备通信。


bus_type的作用：
打印机设备被识别为连接到USB总线（bus_type）。每种总线类型有其特定的通信协议和操作方式，USB总线也有其特定的方式来处理数据传输和设备通信。

设备驱动处理请求：
一旦确定了正确的驱动程序，操作系统通过这个驱动程序向打印机发送数据。驱动程序知道如何将数据格式化和传输到USB总线上的特定设备。
驱动程序可能需要将应用程序的数据转换成打印机能理解的格式，例如打印机控制语言。


数据通过总线传输到设备：
经过驱动程序处理后的数据通过USB总线发送给打印机。USB总线的协议和控制器管理数据的物理传输。


设备响应：
打印机接收到数据，处理并开始打印。
如果有必要，打印机或其驱动程序可能会向操作系统发送状态信息或打印完成的确认。

总结
在这个过程中，bus_type（USB总线）定义了如何物理连接和通信，device（打印机）是实际的硬件设备，而device_driver是使打印机能与操作系统交流的软件组件。这个流程涵盖了数据从应用程序到硬件设备的完整路径，展示了这三个内核组件如何协同工作以实现设备通信和操作。

**以一个USB闪存驱动器为例来说明/dev/目录下的设备文件和内核中的device实例之间的关系**

示例场景
假设你将一个USB闪存驱动器插入计算机的USB端口。


**内核中的device实例的创建**

USB闪存驱动器被连接：
当你插入USB闪存驱动器时，USB总线控制器检测到了一个新设备的连接。

内核识别新设备：
内核的USB子系统响应这个新设备，启动一个新的device实例来代表这个USB闪存驱动器。

加载设备驱动：
根据USB闪存驱动器的识别信息（如厂商ID和产品ID），内核选择并加载适当的驱动程序（如果尚未加载）。
/dev/目录下的设备文件的创建

udev创建设备文件：
一旦内核识别并准备好设备，udev（设备管理器）将在/dev/目录下为该设备创建一个设备文件。假设文件被命名为/dev/sdb1。
应用程序通过设备文件访问USB闪存驱动器


应用程序访问设备：
一个文件浏览器或命令行工具现在可以通过访问/dev/sdb1设备文件来与USB闪存驱动器交互。

读写操作：
当你尝试从USB闪存驱动器读取文件或向其写入文件时，这些操作会转化为对应的read或write系统调用，它们作用于/dev/sdb1。
系统调用经由设备文件转发到内核中负责该设备的驱动程序。
驱动程序随后会操作内核中的device实例来执行实际的硬件操作。


总结
在这个例子中，内核中的device实例是USB闪存驱动器在内核设备模型中的表示，而/dev/sdb1是一个设备文件，它提供了一个用户空间应用程序可以通过文件I/O操作访问USB闪存驱动器的接口。这两者通过设备驱动程序连接起来，使得用户空间操作能够转化为内核和硬件层面的动作。


sysfs is usually mounted in /sys
- `/sys/bus/` contains the list of buses
- `/sys/devices/` contains the list of devices
- `/sys/class` enumerates devices by the framework they are registered to (net, input, block...), whatever bus they are connected to. Very useful!

include/linux/device.h
```c
struct bus_type {
    const char              *name;
    const char              *dev_name;
    struct device           *dev_root;
    const struct attribute_group **bus_groups;
    const struct attribute_group **dev_groups;
    const struct attribute_group **drv_groups;

    int (*match)(struct device *dev, struct device_driver *drv);
    int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
    int (*probe)(struct device *dev);
    int (*remove)(struct device *dev);
    void (*shutdown)(struct device *dev);

    int (*online)(struct device *dev);
    int (*offline)(struct device *dev);

    int (*suspend)(struct device *dev, pm_message_t state);
    int (*resume)(struct device *dev);

    int (*num_vf)(struct device *dev);

    int (*dma_configure)(struct device *dev);

    const struct dev_pm_ops *pm;

    const struct iommu_ops *iommu_ops;

    struct subsys_private *p;
    struct lock_class_key lock_key;

    bool need_parent_lock;
};
```

```c
struct device {
    struct kobject kobj;
    struct device           *parent;

    struct device_private   *p;

    const char              *init_name; /* initial name of the device */
    const struct device_type *type;

    struct bus_type *bus;           /* type of bus device is on */
    struct device_driver *driver;   /* which driver has allocated this
                                        device */
    void            *platform_data; /* Platform specific data, device
                                        core doesn't touch it */
    void            *driver_data;   /* Driver data, set and get with
                                        dev_set_drvdata/dev_get_drvdata */
...

    void    (*release)(struct device *dev);
```

```c
struct device_driver {
    const char              *name;
    struct bus_type         *bus;

    struct module           *owner;
    const char              *mod_name;      /* used for built-in modules */

    bool suppress_bind_attrs;       /* disables bind/unbind via sysfs */
    enum probe_type probe_type;

    const struct of_device_id       *of_match_table;
    const struct acpi_device_id     *acpi_match_table;

    int (*probe) (struct device *dev);
    int (*remove) (struct device *dev);
    void (*shutdown) (struct device *dev);
    int (*suspend) (struct device *dev, pm_message_t state);
    int (*resume) (struct device *dev);
    const struct attribute_group **groups;
    const struct attribute_group **dev_groups;

    const struct dev_pm_ops *pm;
    void (*coredump) (struct device *dev);

    struct driver_private *p;
};
```
总线中既定义了设备，也定义了驱动；设备中既有总线，也有驱动；驱动中既有总线也有设备相关的信息。那这三个的关系到底是什么呢？

三者的关系是[^bus_type_device_device_driver]：

内核要求每次出现一个设备，就要向总线汇报，也叫注册；每次出现一个驱动，也要向总线汇报，或者叫注册。比如系统初始化时，会扫描连接了哪些设备，并为每一个设备建立一个`struct device`变量，并为每一个驱动程序准备一个`struct device_driver`结构的变量。把这些量变量加入相应的链表，形成一条设备链表和一条驱动量表。这样，总线就能通过总线找到每一个设备和每一个驱动程序。

当一个`struct device`诞生，总线就会去driver链表找设备对应的驱动程序。如果找到就执行设备的驱动程序，否则就等待。反之亦然。


register函数的函数原型
include/linux/pci.h
```c
/* pci_register_driver() must be a macro so KBUILD_MODNAME can be expanded */
#define pci_register_driver(driver)     \
    __pci_register_driver(driver, THIS_MODULE, KBUILD_MODNAME)
```

drivers/pci/pci-driver.c
```c
int __pci_register_driver(struct pci_driver *drv, struct module *owner,
              const char *mod_name)
{
    /* initialize common driver fields */
    drv->driver.name = drv->name;
    drv->driver.bus = &pci_bus_type;
    drv->driver.owner = owner;
    drv->driver.mod_name = mod_name;
    drv->driver.groups = drv->groups;

    spin_lock_init(&drv->dynids.lock);
    INIT_LIST_HEAD(&drv->dynids.list);

    /* register with core */
    return driver_register(&drv->driver);
}
```
上面第一个参数是`struct pci_driver`, 下面是它的原型
include/linux/pci.h
```c
struct pci_driver {
    struct list_head    node;
    const char      *name;
    const struct pci_device_id *id_table;   /* Must be non-NULL for probe to be called */
    int  (*probe)(struct pci_dev *dev, const struct pci_device_id *id); /* New device inserted */
    void (*remove)(struct pci_dev *dev);    /* Device removed (NULL if not a hot-plug capable driver) */
    int  (*suspend)(struct pci_dev *dev, pm_message_t state);   /* Device suspended */
    int  (*suspend_late)(struct pci_dev *dev, pm_message_t state);
    int  (*resume_early)(struct pci_dev *dev);
    int  (*resume)(struct pci_dev *dev);    /* Device woken up */
    void (*shutdown)(struct pci_dev *dev);
    int  (*sriov_configure)(struct pci_dev *dev, int num_vfs); /* On PF */
    const struct pci_error_handlers *err_handler;
    const struct attribute_group **groups;
    struct device_driver    driver;
    struct pci_dynids   dynids;
};
```
其中一个成员是我们上面的总线设备模型中的driver的结构体。 其实在linux内核中，所有设备的驱动的定义，都是以struct device_driver为基类，进行继承与扩展的。 你没有看错，**内核当中使用了很多OO的思想**。


现在我们知道了pci设备的驱动程序的描述方法。但是问题又来了：这么复杂的一个结构体，我们怎么用呢？
首先看下实例代码中是怎么玩的：
drivers/net/ethernet/intel/e1000e/netdev.c
```c
/* PCI Device API Driver */
static struct pci_driver e1000_driver = {
    .name     = e1000e_driver_name,
    .id_table = e1000_pci_tbl,
    .probe    = e1000_probe,
    .remove   = e1000_remove,
    .driver   = {
        .pm = &e1000_pm_ops,
    },
    .shutdown = e1000_shutdown,
    .err_handler = &e1000_err_handler
};
```
我们可以看出来，并不是这个结构体的所有成员我们都要操作，我们只管其中最关键的几个就行了。



### USB Bus例子
Core infrastructure (bus driver)
- `drivers/usb/core/`
- `struct bus_type` is defined in `drivers/usb/core/driver.c` and registered in `drivers/usb/core/usb.c`

Adapter drivers
- `drivers/usb/host/`
- For EHCI, UHCI, OHCI, XHCI, and their implementations on various systems
(Microchip, IXP, Xilinx, OMAP, Samsung, PXA, etc.)

Device drivers
- Everywhere in the kernel tree, classified by their type (Example: `drivers/net/usb/`)


The `MODULE_DEVICE_TABLE()` macro allows depmod (run by `make modules_install`)
to extract the relationship between device identifiers and drivers,
so that drivers can be loaded automatically by udev. See
`/lib/modules/$(uname -r)/modules.{alias,usbmap}`

```c
/* table of devices that work with this driver */
static struct usb_device_id rtl8150_table[] = {
    { USB_DEVICE(VENDOR_ID_REALTEK, PRODUCT_ID_RTL8150) },
    { USB_DEVICE(VENDOR_ID_MELCO, PRODUCT_ID_LUAKTX) },
    { USB_DEVICE(VENDOR_ID_MICRONET, PRODUCT_ID_SP128AR) },
    { USB_DEVICE(VENDOR_ID_LONGSHINE, PRODUCT_ID_LCS8138TX) },
    [...]
    {}
};
MODULE_DEVICE_TABLE(usb, rtl8150_table);
```

`struct usb_driver` is a structure defined by the USB core. Each USB device
driver must instantiate it, and register itself to the USB core using this structure

This structure inherits from `struct device_driver`, which is defined by the
device model.

```c
static struct usb_driver rtl8150_driver = {
    .name = "rtl8150",
    .probe = rtl8150_probe,
    .disconnect = rtl8150_disconnect,
    .id_table = rtl8150_table,
    .suspend = rtl8150_suspend,
    .resume = rtl8150_resume
};
```


When the driver is loaded/unloaded, it must register/unregister itself to/from the USB core
Done using `usb_register()` and `usb_deregister()`, provided by the USB core.
```c
static int __init usb_rtl8150_init(void)
{
    return usb_register(&rtl8150_driver);
}
static void __exit usb_rtl8150_exit(void)
{
    usb_deregister(&rtl8150_driver);
}

module_init(usb_rtl8150_init);
module_exit(usb_rtl8150_exit);
```

All this code is actually replaced by a call to the `module_usb_driver()` macro:
```c
module_usb_driver(rtl8150_driver);
```


Registering an I2C device driver: example
```c
static const struct i2c_device_id adxl345_i2c_id[] = {
    { "adxl345", ADXL345 },
    { "adxl375", ADXL375 },
    { }
};
MODULE_DEVICE_TABLE(i2c, adxl345_i2c_id);
static const struct of_device_id adxl345_of_match[] = {
    { .compatible = "adi,adxl345" },
    { .compatible = "adi,adxl375" },
    { },
};
MODULE_DEVICE_TABLE(of, adxl345_of_match);
static struct i2c_driver adxl345_i2c_driver = {
    .driver = {
        .name = "adxl345_i2c",
        .of_match_table = adxl345_of_match,
    },
    .probe = adxl345_i2c_probe,
    .remove = adxl345_i2c_remove,
    .id_table = adxl345_i2c_id,
};
module_i2c_driver(adxl345_i2c_driver);
```


cdev[^cdev] 结构体

include/linux/cdev.h
```c
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
} __randomize_layout;
```

include/linux/fs.h
```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    __poll_t (*poll) (struct file *, struct poll_table_struct *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    int (*check_flags)(int);
    void (*show_fdinfo)(struct seq_file *m, struct file *f);

    ...

} __randomize_layout;
```
提供给应用层操作方法集


include/linux/kdev_t.h

```c
#define MINORBITS       20
#define MINORMASK       ((1U << MINORBITS) - 1)

#define MAJOR(dev)      ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev)      ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi)    (((ma) << MINORBITS) | (mi))
```

dev_t 设备号
用来唯一标识设备的，32位无符号整型。主设备号+从设备号

```c
MAJOR(dev_t dev)        // 从设备号中提取主设备号
MINOR(dev_t dev)        // 从设备号中提取次设备号
MKDEV(int ma, int mi)   // 根据主次设备号生成设备号


#define MAJOR(dev)      ((unsigned int) ((dev) >> 20))
// 32位无符号数右移20位，得到高12位

#define MINOR(dev)      ((unsigned int) ((dev) & ((1U << 20) - 1)))
// 无符号的1 左移20位，第21位为1其余都为0， -1操作后21位为0，1~20位都为1
// 与操作之后 取得低20位
```

**设备号是由高12位的主设备号与低20位的次设备号组成**

---
## 例子
编写一个功能完整的Linux驱动程序比较复杂，但我可以提供一个简化的字符设备驱动程序例子。此驱动程序创建一个设备文件`/dev/example`，用户空间程序可以通过读取和写入这个文件与驱动程序交互。为了简洁，我将省略错误处理和一些边界条件的检查。

首先，你需要具备Linux内核开发的基础知识，包括如何编写Makefile来编译模块以及如何加载和卸载模块。

这里是一个简单的字符设备驱动的例子：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/device.h>

#define DEVICE_NAME "mychardev"
#define CLASS_NAME "chardev"

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple character device driver");
MODULE_VERSION("0.1");

static int majorNumber;
static char message[256] = {0};
static short size_of_message;
static struct class* charClass = NULL;
static struct device* charDevice = NULL;

// 声明文件操作函数
static int dev_open(struct inode *inodep, struct file *filep){
   printk(KERN_INFO "Device opened\n");
   return 0;
}

static ssize_t dev_read(struct file *filep, char *buffer, size_t len, loff_t *offset){
   int err_count = 0;
   err_count = copy_to_user(buffer, message, size_of_message);
   if(err_count==0){
      printk(KERN_INFO "Sent %d characters to the user\n", size_of_message);
      return (size_of_message=0);
   }
   else {
      printk(KERN_INFO "Failed to send %d characters to the user\n", err_count);
      return -EFAULT;
   }
}

static ssize_t dev_write(struct file *filep, const char *buffer, size_t len, loff_t *offset){
   sprintf(message, "%s", buffer);
   size_of_message = strlen(message);
   printk(KERN_INFO "Received %zu characters from the user\n", len);
   return len;
}

static int dev_release(struct inode *inodep, struct file *filep){
   printk(KERN_INFO "Device closed\n");
   return 0;
}

// 定义文件操作结构体
static struct file_operations fops = {
   .open = dev_open,
   .read = dev_read,
   .write = dev_write,
   .release = dev_release,
};

// 初始化模块
static int __init chardev_init(void){
   printk(KERN_INFO "CharDev: Initializing the CharDev LKM\n");
   // 分配主设备号
   majorNumber = register_chrdev(0, DEVICE_NAME, &fops);
   if (majorNumber<0){
      printk(KERN_ALERT "CharDev failed to register a major number\n");
      return majorNumber;
   }
   printk(KERN_INFO "CharDev: registered correctly with major number %d\n", majorNumber);

   // 注册设备类
   charClass = class_create(THIS_MODULE, CLASS_NAME);
   if (IS_ERR(charClass)){
      unregister_chrdev(majorNumber, DEVICE_NAME);
      printk(KERN_ALERT "Failed to register device class\n");
      return PTR_ERR(charClass);
   }
   printk(KERN_INFO "CharDev: device class registered correctly\n");

   // 注册设备驱动
   charDevice = device_create(charClass, NULL, MKDEV(majorNumber, 0), NULL, DEVICE_NAME);
   if (IS_ERR(charDevice)){
      class_destroy(charClass);
      unregister_chrdev(majorNumber, DEVICE_NAME);
      printk(KERN_ALERT "Failed to create the device\n");
      return PTR_ERR(charDevice);
   }
   printk(KERN_INFO "CharDev: device class created correctly\n");
   return 0;
}

// 清理模块
static void __exit chardev_exit(void){
   device_destroy(charClass, MKDEV(majorNumber, 0));
   class_unregister(charClass);
   class_destroy(charClass);
   unregister_chrdev(majorNumber, DEVICE_NAME);
   printk(KERN_INFO "CharDev: Goodbye from the CharDev LKM!\n");
}

module_init(chardev_init);
module_exit(chardev_exit);
```

在上面的代码中，我们定义了一个字符设备驱动程序，它包含了初始化和卸载函数，以及文件操作的实现。这个驱动程序在加载时会注册一个设备和相应的设备类，并且会创建一个设备文件`/dev/example`。

为了编译这个驱动程序，你需要一个Makefile，如下所示：

```makefile
obj-m += example.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

编译驱动程序：

```sh
make
```

加载驱动程序到内核：

```sh
sudo insmod example.ko
```

现在你应该能在`/dev`目录下找到`example`设备文件。你可以使用诸如`cat`和`echo`这样的命令与设备交互。

当你完成测试后，可以卸载驱动程序：

```sh
sudo rmmod example
```

请注意，这个例子是为了说明Linux驱动模型的基础知识，实际的驱动开发可能会更加复杂，需要考虑更多的错误处理、并发控制和系统兼容性问题。




[^bootlin]: https://bootlin.com/doc/training/linux-kernel/linux-kernel-slides.pdf
[^hello.c]: https://github.com/bootlin/training-materials/blob/master/code/hello/hello.c
[^__init]: https://github.com/TongxinV/oneBook/blob/master/0.5.Linux-Driver%20Development/0.0.%E5%AD%97%E7%AC%A6%E8%AE%BE%E5%A4%87%E5%9F%BA%E7%A1%80.md
[^bus_type_device_device_driver]: https://zhuanlan.zhihu.com/p/590369703
[^cdev]: https://www.bilibili.com/video/BV16J411b7pe?p=7



