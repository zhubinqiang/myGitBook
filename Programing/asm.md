# 汇编入门

[TOC]

作者：[不吃油条](https://www.zhihu.com/people/hackeris/posts) 
https://www.zhihu.com/column/c_144694924

## 准备环境
在ubuntu的机器上
```bash
sudo apt install -y gcc yasm nasm
```

## 第一个程序
一个简单的c程序
```c
int main() {
    return 0;
}
```

其等价的汇编代码如下：
```armasm
global main

main:
    mov eax, 0
    ret
```

编译与链接
```bash
yasm -f elf64 -o first.o 1.asm
gcc first.o -o first
```

执行
```bash
./first
echo $?
```

## 1 + 2 = 3
```armasm
global main

main:
    mov eax, 1
    mov ebx, 2
    add eax, ebx
    ret
```
上面的 eax，ebx都是寄存器。

```armasm
main:
    mov eax, 1
    add eax, 2
    add eax, 3
    add eax, 4
    add eax, 5
    ret
```
上面是 1 + 2 + 3 + 4 + 5 = 15

指令
mov: 数据传送指令，我们可以像下面这样用mov指令，达到数据传送的目的。
```armasm
mov eax, 1          ; 让eax的值为1（eax = 1）
mov ebx, 2          ; 让ebx的值为2（ebx = 2）
mov ecx, eax        ; 把eax的值传送给ecx（ecx = eax）
```

add: 加法指令
```armasm
add eax, 2          ; eax = eax + 2
add ebx, eax        ; ebx = ebx + eax
```

ret: 返回指令，类似于C语言中的return，用于函数调用后的返回。


## x86汇编的两种语法
Intel语法 和 AT&T语法

Windows 用Intel语法， Unix使用的是AT&T语法

AT&T 语法
```armasm
movl $1, %eax
```

上面的指令的意思是 CPU 内部产生一个立即数1, 然后传送到 eax 寄存器。

AT&T语法要求:
mov后面的 l 表示 long 
**立即数前面要加 `$`**
**寄存器前要加 `%`**，以便和符号名区分开。
指令 源，目标

Intel 语法
```armasm
mov eax, 1
```

