# 编译 kernel module
[TOC]

## 什么是内核模块
Loadable Kernel Modules（LKM）即可加载内核模块，LKM可以动态地加载到内存中，无须重新编译内核。所以它经常被用于一些设备的驱动程序，例如声卡，网卡等等[^1]。

内核模块和一般的 C 语言程序不同，它不使用 main() 函数作为入口，并且有如下区别：

- 非顺序执行：内核模块使用初始化函数来进行注册，并处理请求，初始化函数运行后就结束了。 它可以处理的请求类型在模块代码中定义。
- 没有自动清理：内核模块申请的所有内存，必须要在模块卸载时手动释放，否则这些内存会无法使用，直到重启，也就是说我们需要在模块的卸载函数（也就是下文写到的退出函数）中，将使用的内存逐一释放。
- 会被中断：内核模块可能会同时被多个程序/进程使用，构建内核模块时要确保发生中断时行为一致和正确。想了解更多请看：Linux 内核的[中断机制](https://www.cnblogs.com/linfeng-learning/p/9512866.html)
- 更高级的执行特权：通常分配给内核模块的CPU周期比分配给用户空间程序的要多。编写内核模块时要小心，以免模块对系统的整体性能产生负面影响。
- 不支持浮点：在Linux内核里无法直接进行浮点计算，因为这样做可以省去在用户态与内核态之间进行切换时保存/恢复浮点寄存器 FPU的操作。

## 准备
```sh
sudo yum install -y \
    kernel-devel \
    kernel-headers \
    gcc \
    make \
    bc \
    openssl-devel \
    elfutils-libelf-devel \
    bison \
    flex
```

- kernel-devel: kernel 代码。
- kernel-headers: kernel 头文件。
- [elfutils](https://sourceware.org/elfutils/): (**E**xecutable and **L**inksing **F**ormat) 用来读写ELF文件的。
- [bison](https://www.gnu.org/software/bison/): 用于自动生成语法分析器程序。
- [flex](https://www.gnu.org/software/flex/): 词法分析器生成器。

## 代码
sample_module.c:
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("bzhux");
MODULE_DESCRIPTION("A simple Linux driver to say hello.");
MODULE_VERSION("0.1");

int sample_module_init(void) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
    return 0;
}

void sample_module_exit(void) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
}

module_init(sample_module_init);
module_exit(sample_module_exit);
```

Makefile文件[^2]:
```makefile
KDIR=/lib/modules/$(shell uname -r)/build
obj-m += sample_module.o

all:
    make -C $(KDIR) M=$(PWD) modules

clean:
    make -C $(KDIR) M=$(PWD) clean
```

-C : 在执行前先切换到 DIRECTORY 目录
M= : Makefile的目录to specify directory of external module to build

## 执行
编译kernel module
```sh
make
```

生成如下：
```sh
$ ls -1
Makefile
modules.order
Module.symvers
sample_module.c
sample_module.ko
sample_module.mod.c
sample_module.mod.o
sample_module.o
```

安装模块
```sh
sudo insmod sample_module.ko
```

查看模块
```sh
$ modinfo sample_module.ko
filename:       /home/bzhux/WS/try/kernel/sample_module.ko
version:        0.1
description:    A simple Linux driver to say hello.
author:         bzhux
license:        GPL
srcversion:     60CD8F4CC9E4CEF3EB452BE
depends:
retpoline:      Y
name:           sample_module
vermagic:       5.0.0-38-generic SMP mod_unload
```

删除模块
```sh
sudo rmmod sample_module
```

dmesg查看输出
```sh
$ dmesg
[6678778.363669] inside the sample_module_exit function
[6678892.946450] inside the sample_module_init function
```

## 实现 file_operations
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/fs.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("bzhux");
MODULE_DESCRIPTION("A simple Linux driver to say hello.");
MODULE_VERSION("0.1");

int sample2_open(struct inode *pinode, struct file *pfile) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
    return 0;
}

ssize_t sample2_read(struct file *pfile, char __user *buffer, size_t length, loff_t *offset) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
    return length;
}

ssize_t sample2_write(struct file *pfile, const char __user *buffer, size_t length, loff_t *offset) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
    return length;
}

int sample2_close(struct inode *pinode, struct file *pfile) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);
    return 0;
}

/* to hold the file opreations performed on this device
 * refer to /lib/modules/$(uname -r)/build/include/linux/fs.h
 * */
struct file_operations sample2_file_operations = {
    .owner = THIS_MODULE,
    .open = sample2_open,
    .read = sample2_read,
    .write = sample2_write,
    .release = sample2_close
};

int sample2_module_init(void) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);

    /* Register with the kernel and indicate that we are registering a character device driver */
    register_chrdev(240, /* Major number */
            "Sample-Char-Drv", /* Name of the driver */
            &sample2_file_operations /* file operations */
            );
    return 0;
}

void sample2_module_exit(void) {
    printk(KERN_ALERT "inside the %s function\n", __FUNCTION__);

    /* unregister the character device dirver */
    unregister_chrdev(240, "Sample-Char-Drv");
}

module_init(sample2_module_init);
module_exit(sample2_module_exit);
```

## 参考
[^1]: https://limxw.com/posts/linux-kernel-practice-hello/
[^2]: http://derekmolloy.ie/writing-a-linux-kernel-module-part-1-introduction/

