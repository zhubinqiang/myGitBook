# GPG

[TOC]

GPG [^gunpg] 是 GNU Privacy Guard.


## 安装gpg
```bash
sudo apt install -y gpg
```

## 生成密钥
```bash
gpg --generate-key

gpg --full-generate-key
```

使用 `--generate-key` [^generate-key] 参数可以创建一个使用默认值的密钥对，
如果想设置更多的值可以使用 --full-generate-key 参数
生成[^generate]密钥使用默认的选项就可以了。


通过 `gpg --list-public-keys` 和 `gpg --list-secret-keys` 来查看公钥和私钥
```
root@b496e1758b3f:/tmp/GPG# gpg --list-public-keys
/root/.gnupg/pubring.kbx
------------------------
pub   rsa3072 2022-11-01 [SC]
      858D892C0390D2184076DDAEB52295F79F580CB9
uid           [ultimate] admin (admin's pgp) <admin@example.com>
sub   rsa3072 2022-11-01 [E]

root@b496e1758b3f:/tmp/GPG# gpg --list-secret-keys
/root/.gnupg/pubring.kbx
------------------------
sec   rsa3072 2022-11-01 [SC]
      858D892C0390D2184076DDAEB52295F79F580CB9
uid           [ultimate] admin (admin's pgp) <admin@example.com>
ssb   rsa3072 2022-11-01 [E]
```

## 加密与解密
加密过程
```bash
date > data.txt

gpg --encrypt --output data.gpg --recipient admin@example.com data.txt

## 生成 data.txt.gpg
gpg -er 858D892C0390D2184076DDAEB52295F79F580CB9 data.txt

## 指定输出文件
gpg -er 858D892C0390D2184076DDAEB52295F79F580CB9 -o data.encrypt data.txt

## 将文件的md5值加密
md5sum data.txt | awk '{print $1}' | gpg -aer 858D892C0390D2184076DDAEB52295F79F580CB9
```

`-e`, `--encrypt`: 加密
`-r`, `--recipient`: 指定密钥

解密
```bash
gpg --decrypt --output message.txt data.gpg

gpg -du 858D892C0390D2184076DDAEB52295F79F580CB9 data.txt.gpg 2> /dev/null
```

`-d`, `--decrypt`: 解密
`-u`, `--local-user`: 指定密钥

## 签名与校验

二进制签名
```bash
gpg -u 858D892C0390D2184076DDAEB52295F79F580CB9 --sign data.txt
```

文本签名
```bash
## 生成 data.txt.asc
gpg -u 858D892C0390D2184076DDAEB52295F79F580CB9 --clear-sign data.txt
```

分离式签名
```bash
# 二进制分离式签名
gpg -u 858D892C0390D2184076DDAEB52295F79F580CB9 -b -o data.bin.sig data.txt

# 文本分离式签名
gpg -u 858D892C0390D2184076DDAEB52295F79F580CB9 -ab -o data.asc.sig data.txt
```

`-b`, `--detach-sign` 源内容与签名存放在不同文件，签名与源文件可以分别发布，默认签名为二进制形式，默认输出文件名为 xxx.sig。
`-a`, `--armor` 文本形式的分离式签名 此时默认输出文件名为 xxx.asc

签名并加密
```bash
gpg -u 858D892C0390D2184076DDAEB52295F79F580CB9 -r 858D892C0390D2184076DDAEB52295F79F580CB9 -se data.txt
```

验证
```bash
gpg --verify data.txt.gpg

gpg --verify data.bin.sig data.txt
gpg --verify  data.asc.sig  data.txt
```


## 导出与导入密钥
导出之前先签名，我这里已经签好了。
```
root@b496e1758b3f:/dev/shm/GPG# gpg --edit-key 858D892C0390D2184076DDAEB52295F79F580CB9
gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa3072/B52295F79F580CB9
     created: 2022-11-01  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa3072/023CF254594F642A
     created: 2022-11-01  expires: never       usage: E
[ultimate] (1). admin (admin's pgp) <admin@example.com>

gpg> fpr
pub   rsa3072/B52295F79F580CB9 2022-11-01 admin (admin's pgp) <admin@example.com>
 Primary key fingerprint: 858D 892C 0390 D218 4076  DDAE B522 95F7 9F58 0CB9

gpg> sign
"admin (admin's pgp) <admin@example.com>" was already signed by key B52295F79F580CB9
Nothing to sign with key B52295F79F580CB9

gpg> q
```

```bash
gpg --output admin-key.gpg --export 858D892C0390D2184076DDAEB52295F79F580CB9
```


在另一台机器上面
```bash
gpg --import admin-key.gpg
gpg --list-public-keys
```


[^gunpg]: https://gnupg.org/
[^generate-key]: https://blog.ginshio.org/2020/gpg_started_guide/ 作者：GinShio
[^generate]: https://linux.cn/article-14082-1.html 作者：Hunter Wittenborn 译者：Xingyu.Wang



