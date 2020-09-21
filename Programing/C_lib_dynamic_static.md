# C语言动态库与静态库

[TOC]

## 动态库
so_test.h
```c
void fa();
void fb();
void fc();
```

a.c
```c
#include <stdio.h>
#include <so_test.h>

void fa() {
    printf("AAAAA\n");
}
```

b.c
```c
#include <stdio.h>
#include <so_test.h>

void fb() {
    printf("BBBBB\n");
}
```

c.c
```c
#include <stdio.h>
#include <so_test.h>

void fc() {
    printf("CCCCC\n");
}
```

### 编译生成动态库
```sh
gcc a.c b.c c.c -I. -fPIC -shared -o libtest.so
```
PIC: position independent code

这步能编译出 libXXX.so.x
```sh
gcc a.c b.c c.c -I .-fPIC -shared -Wl,-soname,libtest.so.2 -o libtest.so.2.1

ln -sf libtest.so.2.1 libtest.so.2
ln -sf libtest.so.2 libtest.so
```

```sh
$ readelf -d libtest.so | grep soname
0x000000000000000e (SONAME)             Library soname: [libtest.so.2]
```

### 加入动态库进行编译
dynamic.c
```c
#include <stdio.h>
#include <so_test.h>

int main(void) {
    fa();
    fb();
    fc();
}
```

```sh
gcc dynamic.c -I . -L . -ltest -o dynamic
```

> `-I` 告诉编译器去哪里找头文件
> `-L` 去哪里找库文件, **即使库文件在当前目录，编译器也不会主动去找的**。
> `-ltest` 告诉编译器要使用 libtest.so 或者 libtest.a

也可以通过环境变量指定
```sh
export C_INCLUDE_PATH=/path/to/include:${C_INCLUDE_PATH}
export CPP_INCLUDE_PATH=/path/to/include:${CPP_INCLUDE_PATH}
```

### 运行带有动态库的可执行文件
执行的时候要导入到 `LD_LIBRARY_PATH` 变量中。否则不一定能找到，会报"cannot open shared object file"  
```sh
export LD_LIBRARY_PATH=.
./dynamic
```

上面的是临时执行，永久生效可以把路径写入`/etc/ld.so.conf` 或 `/etc/ld.so.conf.d/xxx.conf` 
再或者把动态库拷贝到默认找寻的动态库目录像`/usr/lib64`
再后执行 `ldconfig`

也可以使用 `patchelf` 命令指定
```sh
patchelf --set-rpath $PWD ./dynamic
```

## 编译静态库
先生成`.o`的目标文件
```sh
gcc -I . -c a.c
gcc -I . -c b.c
gcc -I . -c c.c
```

### 生成静态库
```sh
ar -crv libtest.a a.o b.o c.o
```

### 加入静态库进行编译
static.c
```c
#include <stdio.h>
#include <so_test.h>

int main(void) {
    fc();
    fb();
    fa();
}
```

```sh
gcc static.c -L . -I . -ltest -o static
```

### 运行带有静态库的可执行文件
```sh
./static
```

## 注意事项
1. 动态库和静态库同名时，优先选择动态库。
2. 动态库文件名的命名规范是在动态库名增加前缀lib，但其文件扩展名为.so
3. 静态库文件名的命名规范是以lib为前缀，紧接着跟静态库名，扩展名为.a

## makefile
```makefile
INC = -I./include
SRC = ./src

run: main
        ./main

main: run.c a.o b.o c.o
        cc $(INC) -o $@ $^

%.o: $(SRC)/%.c
        cc $(INC) -c $^

.PHONY: clean run main libs1 libs2 static dynamic

clean:
        rm -rf *.o *.so run main dynamic static

libs1: $(SRC)/a.c $(SRC)/b.c $(SRC)/c.c
        cc $(INC) -o libtest.so -fPIC -shared $^

libs2: a.o b.o c.o
        ar -crv libtest.a $^

static: libs2 run.c
        cc run.c $(INC) -L. -ltest -o $@

dynamic: libs1 run.c
        cc run.c $(INC) -L. -ltest -o $@
```