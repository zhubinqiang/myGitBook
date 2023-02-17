# NFS
[TOC]

参考[这里](https://www.cnblogs.com/lykyl/archive/2013/06/14/3136921.html)

## 安装
RHEL/CentOS
```sh
sudo yum install -y nfs-utils portmap
```

Debian/Ubuntu
```sh
sudo apt-get install -y nfs-kernel-server
```

## 配置
配置文件在 /etc/exports
```
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)

/datadisk   *(rw,sync,no_root_squash,no_subtree_check)
```

格式 <共享目录> 客户端1(选项) [客户端2(选项) ...]

### 客户端
指定IP: 192.168.1.100
子网IP: 192.168.1.100/24  或者 192.168.1.100/255.255.255.0
指定域名： nfs.test.com
域中的所有主机: *.test.com
所有主机: *

### 选项
ro：共享目录只读；
rw：共享目录可读可写；
all_squash：所有访问用户都映射为匿名用户或用户组；
no_all_squash（默认）：访问用户先与本机用户匹配，匹配失败后再映射为匿名用户或用户组；
root_squash（默认）：将来访的root用户映射为匿名用户或用户组；
no_root_squash：来访的root用户保持root帐号权限；
anonuid=<UID>：指定匿名访问用户的本地用户UID，默认为nfsnobody（65534）；
anongid=<GID>：指定匿名访问用户的本地用户组GID，默认为nfsnobody（65534）；
secure（默认）：限制客户端只能从小于1024的tcp/ip端口连接服务器；
insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
wdelay（默认）：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率；
no_wdelay：若有写操作则立即执行，应与sync配合使用；
subtree_check（默认） ：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限；
no_subtree_check ：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；

## NFS相关命令
`exportfs`: 不重启nfs服务应用更新
`-a` 全部挂载或卸载 /etc/exports中的内容
`-r` 重新读取/etc/exports 中的信息 ，并同步更新/etc/exports、/var/lib/nfs/xtab
`-u` 卸载单一目录（和-a一起使用为卸载所有/etc/exports文件中的目录）
`-v` 在export的时候，将详细的信息输出到屏幕上。

`nfsstat`: 查看NFS的运行状态

`showmount`: 查询nfs共享目录信息
`-a` 显示已经于客户端连接上的目录信息
`-e` IP或者hostname 显示此IP地址分享出来的目录

```sh
## nfs共享目录情况
showmount -e example.com  

## 共享目录连接情况
showmount -a example.com
```

## 挂载
```sh
sudo mount -t nfs example.com:/datadisk /mnt
```

搭建windows nfs client端，参考[这里](https://odinxu.com/post/windows-access-centos-nfs/)。

`Win + R` --> `OptionalFeatures` --> Services for NFS

选择
1. Administroative Tools
2. client for NFS

然后在终端上面运行:
```powershell
showmount -e example.com
mount \\example.com\datadisk X:
```

