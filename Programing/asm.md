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

## 内存
保存到内存
```armasm
mov [0x5566], eax
```

将eax寄存器的值，保存到编号为0x5566对应的内存里去，
这里我们使用的是4个字节32位的寄存器，一个eax需要4个字节的空间才装得下，
所以编号为 0x5566, 0x5567, 0x5568, 0x5569 这四个字节都会被eax的某一部分覆盖掉。

从内存中读取
```armasm
mov eax, [0x0699]
```
把0x0699这个地址对应那片内存区域中的后4个字节取出来放到eax里面去。


```armasm
global main

main:
    mov ebx, 1
    mov ecx, 2
    add ebx, eax

    mov [0x233], ebx
    mov eax, [0x233]

    ret
```
上面的程序执行出现"Segmentation fault (core dumped)"了。

这里的原因是：**我们的程序运行在一个受管控的环境下，是不能随便读写内存的**。

修改后
```armasm
global main

main:
    mov ebx, 1
    mov ecx, 2
    add ebx, ecx

    mov [sui_bian_xie], ebx
    mov eax, [sui_bian_xie]

    ret

section .data
sui_bian_xie dw 0
```

上面用gcc链接的时候 出现了下面的问题
【relocation R_X86_64_32 against `.data' can not be used when making a PIE object; recompile with -fPIE】

有下面2种解决方法：
1. 根据 [stackoverflow](https://stackoverflow.com/questions/48669436/cannot-compile-nasm-program) 上面给的回答 给gcc 加上参数 `-no-pie`。
但这个不是推荐用法。
2. 根据 [nasm文档](https://www.nasm.us/xdoc/2.10.09/html/nasmdoc6.html#section-6.2) 添加 `DEFAULT REL` 

整理一下代码如下：
```armasm
DEFAULT REL
global main

main:
    mov ebx, 1
    mov ecx, 2
    add ebx, ecx

    mov [sui_bian_xie], ebx
    mov eax, [sui_bian_xie]

    ret

section .data
sui_bian_xie dw 0
```

`section .data`：接下来的内容经过编译后，会放到可执行文件的数据区域，同时也会随着程序启动的时候，分配对应的内存。
`sui_bian_xie dw 0`：开辟一块2字节的空间，并且里面用0填充。这里的dw（define word）就表示2个字节，前面那个sui_bian_xie的意思就是这里可以随便写，也就是起个名字而已，方便自己写代码的时候区分，这个sui_bian_xie会在编译时被编译器处理成一个具体的地址，我们无需理会地址具体时多少，反正知道前后的sui_bian_xie指代的是同一个东西就行了。

- DB - Define Byte. 8 bits[^DB]
- DW - Define Word. Generally 2 bytes on a typical x86 32-bit system
- DD - Define double word. Generally 4 bytes on a typical x86 32-bit system







## x86汇编的两种语法
Intel语法 和 AT&T语法

Windows 用Intel语法， Unix使用的是AT&T语法

AT&T 语法
```armasm
movl $1, %eax
```

上面的指令的意思是 CPU 内部产生一个立即数1, 然后传送到 eax 寄存器。

AT&T[^at]语法要求:
mov后面的 l 表示 long 
**立即数前面要加 `$`**
**寄存器前要加 `%`**，以便和符号名区分开。
指令 源，目标

Intel 语法
```armasm
mov eax, 1
```

[^DB]: https://stackoverflow.com/questions/10168743/which-variable-size-to-use-db-dw-dd-with-x86-assembly
[^at]: https://akaedu.github.io/book/ch18s01.html



