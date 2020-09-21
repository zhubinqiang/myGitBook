# FPM
[TOC]

`fpm` 是一个能快速生成rpm的工具

参考[这里](https://www.cnblogs.com/Roach57/p/5130283.html)

## 安装
```sh
yum install -y ruby ruby-devel
```

使用淘宝的镜像
```sh
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

## 配置 gem
vim ~/.gemrc
```
- https://gems.ruby-china.com
:ssl_verify_mode: 0
```

安装 fpm
```sh
gem install fpm
```

## 运行 fpm
以httplight1.4.32为例

下载源码包 http://download.lighttpd.net/lighttpd/releases-1.4.x/lighttpd-1.4.32.tar.gz

执行脚本
```sh
#!/bin/bash

CWD=$(cd `dirname $0`; pwd)

TARGET=${CWD}/target
SOURECES=${TARGET}/SOURCE
INSTALL=${TARGET}/INSTALL

mkdir -p ${SOURECES}
mkdir -p ${INSTALL}
cp lighttpd-1.4.32.tar.gz ${SOURECES}

cd ${SOURECES}
tar xzvf lighttpd-1.4.32.tar.gz
cd lighttpd-1.4.32
./configure --prefix=/usr --libdir=/usr/lib64
make
make install DESTDIR=${INSTALL}

echo "#!/bin/bash" > ${SOURECES}/after_install.sh
echo "/sbin/ldconfig" >> ${SOURECES}/after_install.sh

echo "#!/bin/bash" > ${SOURECES}/after_remove.sh
echo "/sbin/ldconfig" >> ${SOURECES}/after_remove.sh

## fpm
cd ${CWD}
fpm -f -s dir \
    -t rpm \
    -n lighttpd \
    -v 1.4.32 \
    --iteration 1 \
    -C ${INSTALL} \
    -p ${TARGET} \
    --description 'lighttpd-1.4.32' \
    --url 'http://www.lighttpd.net/' \
    --after-install ${SOURECES}/after_install.sh \
    --after-remove ${SOURECES}/after_remove.sh \
    --vendor  zbq \
    --license BSD \
    --maintainer zbq \
    --category "Development/Tools"
```

这会在 target 目录下生成 lighttpd-1.4.32-1.x86_64.rpm