# Shell 技巧
[TOC]

## 挂载
### 开机挂在iso文件
修改 /etc/fstab
```
/root/centos7.iso /root/mnt udf,iso9660 user,loop 0 0
```

### 挂载目录
```sh
mount --bind /test1 /test2
```

/etc/fstab
```
/test1 /test2 none rw,bind 0 0
```

### 挂载 sshfs
```sh
## 安装
#sudo yum install -y sshfs
#sudo apt install -y sshfs

sshfs user@example.com:/home/user/WS mnt/
```

### 挂载 ntfs
安装 ntfs-3g
```sh
sudo yum install -y ntfs-3g
sudo mount /dev/sdc /mnt
```

### 挂载 exfat
```sh
sudo apt install -y exfat-utils
```

### 挂载 Samba
```sh
sudo mount -t cifs //10.239.128.63/bzhux /mnt -o user=media,passwd=XXX

# 像FTP客户端一样使用smbclient, 不加-U 以当前用户名访问
smbclient //192.168.0.1/tmp  -U username%password

# 查看共享文件夹
smbclient -L 198.168.0.1 -U username%password
```

### 挂载 loopback
```sh
sudo losetup /dev/loop0 VirtualMachineImage.raw
sudo kpartx -a /dev/loop0
sudo kpartx -l /dev/loop0
sudo mount /dev/mapper/loop0p1 /mnt

## umount
sudo unmount /mnt
sudo kpartx -d /dev/loop0
sudo losetup -d /dev/loop0
```

### 挂载虚拟硬盘
```bash
## dd命令创建128M的文件
dd if=/dev/zero bs=1M count=128 of=./vdisk.img

## 创建文件系统
mkfs.ext4 vdisk.img

## 挂载
sudo mount vdisk.img /mnt

## 查看 loop 设备
losetup -a
```

### 挂载虚拟文件系统
参考[这里](https://segmentfault.com/a/1190000006878392)

```bash
sudo mount -t proc none /mnt

sudo mount -t tmpfs -o size=64m none /mnt
```

> proc、tmpfs、sysfs、devpts等都是Linux内核映射到用户空间的虚拟文件系统
> 上面由于没有对应的源设备，上面的none可以是任意字符串。

### 挂载 AUFS
```sh
sudo mount -t aufs -o dirs=./fruits:./vegetables none ./mnt

sudo mount -t aufs -o dirs=./fruits=rw:./vegetables=rw none ./mnt
```

> 如果有重复的文件名，在mount命令行上，越往前的就优先级越高。

### 挂载 overlay
参考: [这里](https://wiki.archlinux.org/index.php/Overlay_filesystem)
```sh
mount -t overlay overlay -o lowerdir=/lower1:/lower2:/lower3,upperdir=/upper,workdir=/work /merged
```

> lowerdir 是readonly的，写操作都发生再 upperdir

上面的样例有这样的顺序:
```
/upper
/lower1
/lower2
/lower3
```

### umount: target is busy
遇到 `umount: /mnt: target is busy.`
```sh
$ fuser -mv /mnt
                     USER        PID ACCESS COMMAND
/mnt:                root     kernel mount /mnt
                     bzhux      7675 ..c.. node
                     bzhux      8018 ..c.. node

$ kill 7675 8018
$ sudo umount /mnt
```

## sudo OR root
### 保存 sudo 密码
```bash
PASS="password"
echo ${PASS} | sudo -S "$@"

# cache 中不保存 sudo密码
sudo -K 
```

### read 输入密码不回显
```sh
stty -echo
read -p "Password:" pass
stty echo
```

### root 或sudo 执行
```bash
PROXY="http://proxy-example.com:911"

export http_proxy=$PROXY
export https_proxy=$PROXY

# rerun the script under sudo if not ran under root
(( EUID != 0 )) && exec sudo -E -- "$0" "$@"
export | grep -i proxy
```
> sudo -E： 保存当前环境变量到sudo中

这样也可以为sudo添加环境变量
```sh
sudo I915_DEV_NAME=${I915_DEV_NAME} LD_LIBRARY_PATH=.:${METRICS_DIR}: ${METRICS_DIR}/metrics_monitor
```

用media用户执行sudo
```sh
sudo -u media mkdir /datadisk
```

### sudo 不用密码
修改/etc/sudoers
#### SUSE, CentOS6
```
media       ALL=(ALL)   NOPASSWD:  ALL
```

#### CentOS7
```
%wheel  ALL=(ALL)       NOPASSWD: ALL
```

> media 用户得要加入wheel组  `usermod -G wheel media`

#### Ubuntu
```
%sudo       ALL=NOPASSWD:  ALL
```

### sshpass 密码登录
```sh
sshpass -p PASSWORD ssh user@example.com mkdir -p /tmp/abc

sshpass -p PASSWORD sudo fdisk -l
```

## sudo 权限不够
### sudo 重定向
```sh
echo 1 | sudo tee /proc/sys/net/ipv4/icmp_echo_ignore_all
```

> `echo 1 | sudo tee -a xyz.txt` 追加到文件

### 将引号内的内容作为一个字符串执行
```sh
sudo sh -c "echo a > 1.txt"
```

> `sudo bash -c "echo a > 1.txt"` 也可以

## Terminal 有颜色
```sh
RCol='\e[0m'    # Text Reset

# Regular           Bold                Underline           High Intensity      BoldHigh Intens     Background          High Intensity Backgrounds
Bla='\e[0;30m';     BBla='\e[1;30m';    UBla='\e[4;30m';    IBla='\e[0;90m';    BIBla='\e[1;90m';   On_Bla='\e[40m';    On_IBla='\e[0;100m';
Red='\e[0;31m';     BRed='\e[1;31m';    URed='\e[4;31m';    IRed='\e[0;91m';    BIRed='\e[1;91m';   On_Red='\e[41m';    On_IRed='\e[0;101m';
Gre='\e[0;32m';     BGre='\e[1;32m';    UGre='\e[4;32m';    IGre='\e[0;92m';    BIGre='\e[1;92m';   On_Gre='\e[42m';    On_IGre='\e[0;102m';
Yel='\e[0;33m';     BYel='\e[1;33m';    UYel='\e[4;33m';    IYel='\e[0;93m';    BIYel='\e[1;93m';   On_Yel='\e[43m';    On_IYel='\e[0;103m';
Blu='\e[0;34m';     BBlu='\e[1;34m';    UBlu='\e[4;34m';    IBlu='\e[0;94m';    BIBlu='\e[1;94m';   On_Blu='\e[44m';    On_IBlu='\e[0;104m';
Pur='\e[0;35m';     BPur='\e[1;35m';    UPur='\e[4;35m';    IPur='\e[0;95m';    BIPur='\e[1;95m';   On_Pur='\e[45m';    On_IPur='\e[0;105m';
Cya='\e[0;36m';     BCya='\e[1;36m';    UCya='\e[4;36m';    ICya='\e[0;96m';    BICya='\e[1;96m';   On_Cya='\e[46m';    On_ICya='\e[0;106m';
Whi='\e[0;37m';     BWhi='\e[1;37m';    UWhi='\e[4;37m';    IWhi='\e[0;97m';    BIWhi='\e[1;97m';   On_Whi='\e[47m';    On_IWhi='\e[0;107m';

echo -e "${Blu}blue ${Red}red ${RCol}etc...."
echo -e "${Red}red${RCol}, ${UGre}green${RCol}, ${Whi}white ${RCol}"
```


```sh
ECHO_PREFIX_INFO="\033[1;32;40mINFO...\033[0;0m"
ECHO_PREFIX_ERROR="\033[1;31;40mError...\033[0;0m"

ECHO_PREFIX_DEF="\e[1;31;40mXXX...\e[0m"

# Try command  for test command result.
function try_command {
    "$@"
    status=$?
    if [ $status -ne 0 ]; then
        echo -e $ECHO_PREFIX_ERROR "ERROR with \"$@\", Return status $status."
        exit $status
    fi
    return $status
}
```

`echo -e "\033[1;32;40mINFO...\033[0;0m`
- `\033`: 这里是八进制的 ESC, 也可以用`\e`
- `\033[`: 控制序列导入器。 (Ctrl+V) + ESC
- `1`: 显示方式。1是加粗
- `32`: 前景色。3开头的都是前景色 32绿色
- `40`: 背景色。4开头的都是背景色 40黑色
- `m`: 在字符串之前
- `\e[m`: 结束
- `\033[0m` : 控制选项. 0m关闭所有属性

[参考于这里](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97)

| 前景色 | 背景色 |  颜色  |
| :----- | ------ | ------ |
| 30     | 40     | 黑色   |
| 31     | 41     | 红色   |
| 32     | 42     | 绿色   |
| 33     | 43     | 黄色   |
| 34     | 44     | 蓝色   |
| 35     | 45     | 紫红色 |
| 36     | 46     | 青蓝色 |
| 37     | 47     | 白色   |

## ssh
### 解决ssh key冲突
Add correct host key in /home/media/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/media/.ssh/known_hosts:8

1. 用ssh-keygen 删除
```bash
ssh-keygen -f "/home/media/.ssh/known_hosts" -R 10.239.173.13 
```

2. 删除known_hosts第8行
```
vim /home/media/.ssh/known_hosts +8
```

### ssh 利用远程机器的key
1.得到要访问的服务器的私钥 id_rsa.bzhux 放到 ~/.ssh/ 下
```sh
cp id_rsa.bzhux ~/.ssh/
chmod 600 ~/.ssh/id_rsa.bzhux
```

新建一个 ~/.ssh/config
```sh
touch ~/.ssh/config
chmod 600 ~/.ssh/config
```

写入内容
```
# bzhux@myhost.com
Host bzhux
  HostName myhost
  Port 22
  User bzhux
  IdentityFile ~/.ssh/id_rsa.bzhux
```

```sh
ssh bzhux@myhost.com
```

### ssh无密码登陆 
方法1
```bash
ssh-keygen
#ssh-copy-id -u user -i ~/.ssh/id_rsa.pub user@host
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
ssh user@host
```

方法2
本地机器 生成ssh key, 在~/.ssh下 生成 ~/.ssh/id_rsa.pub 和 ~/.ssh/id_rsa
```bash
ssh-keygen
```

远程机器
```bash
cd ~/.ssh/
touch authorized_keys
chmod 600 authorized_keys
```

> 1. 把本地机器上的 ~/.ssh/id_res.pub 的内容追加到 远程机器上的 ~/.ssh/authorized_keys
> 2. 估计2种方式是一样的

### 取消弹出信息
Are you sure you want to continue connecting (yes/no)?

修改 /etc/ssh/ssh_config 或者 ~/.ssh/config
```
StrictHostKeyChecking no
```

### 取消写入到 ~/.ssh/known_hosts
修改 /etc/ssh/ssh_config 或者 ~/.ssh/config
```
UserKnownHostsFile /dev/null
```

### 远程执行本地命令
参考:[这里](https://www.cnblogs.com/sparkdev/p/6842805.html)
```sh
## 无参数, 在远程执行本地命令
ssh user@server 'bash -s' < local.sh helloworld

## 本地脚本用绝对路径, 后面可以加参数
ssh user@server /home/user/local.sh helloworld

## 用 : 执行多条命令
ssh user@server "pwd; ls; date"

## 用 -t 交互
ssh -t user@server "sudo fdisk -l"
```


### ssh 本地端口转发
参考[这里](https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/)

所谓本地端口转发，就是**将发送到本地端口的请求，转发到目标端口**。这样，就可以通过访问本地端口，来访问目标端口的服务。使用-L属性，就可以指定需要转发的端口，语法是这样的:

```
ssh -L 本地网卡地址:本地端口:目标地址:目标端口
```

在user1的服务器上执行：
```ssh
ssh -L 10.67.116.24:2222:192.168.100.100:22 user2@192.168.100.123
```

```
+------------+      +---------------+     +---------------+
|10.67.116.24|      |192.168.100.123|     |192.168.100.100|
|2222 user1  +----> |     user2     +---> |22 user3       |
+------------+      +---------------+     +---------------+
```

在client上执行下面ssh命令，就能登录到user3的那台机器
```sh
ssh -p 2222 user3@10.67.116.24
```

### ssh 远程端口转发
远程端口转发，就是**将发送到远程端口的请求，转发到目标端口**。这样，就可以通过访问远程端口，来访问目标端口的服务。使用-R属性，就可以指定需要转发的端口。

在user2的服务器上执行：
```sh
ssh -R 10.67.116.24:8080:192.168.100.100:80 user1@10.67.116.24
```

```
+-------------------------     +---------------+     +---------------+
|http://10.67.116.24/8080|     |192.168.100.123|     |192.168.100.100|
|user1                   | <---+          user2+---> |80        user3|
+------------------------+     +----------------     +---------------+
```

> 远程端口转发 默认只绑定到 localhost上, 通过IP不能访问, 以下方式能解决
            
/etc/ssh/sshd_config 添加这条 并重启ssh服务
```
GatewayPorts clientspecified
```

```sh
sudo systemctl restart sshd.service
# sudo /etc/init.d/ssh restart
```

### ssh 动态端口转发
服务器端：
```sh
## ssh -D local端口 SSH服务器
ssh -D *:1234 10.67.116.24 -N
```

客户端：
```sh
curl -x socks5://10.67.116.24:1234 192.168.100.123
```
上面的 `-x`: 指定代理服务器 通过 SOCKS5 协议访问

```sh
ssh -o ProxyCommand="ncat --proxy-type socks4 --proxy 10.67.116.24:1234 %h %p" user@remote-server

ssh -o ProxyCommand="nc -X 5 -x 10.67.116.94:1234 %h %p" user@remote-server
```

### ssh socks 代理
参考[这里](https://bitbucket.org/gotoh/connect/src/) 编译出 `connect` 命令. 并加入到PATH里面。

在 ~/.ssh/config
```
Host 154.43.32.4
    ProxyCommand connect -S proxy-socks-example.com:1080 %h %p

Host github.com
    ProxyCommand nc -X 5 -x proxy-socks-example.com:1080 %h %p
```

1. nc -X 5: 使用 socks5 协议
2. nc -x: 代理服务器地址

然后在客户端执行
```sh
chmod 644 ~/.ssh/config
ssh -p 4022 user@remote-server
```

或者在终端
```sh
ssh -o ProxyCommand="nc --proxy-type socks4 --proxy proxy-socks-example:1080 %h %p" user@remote-server
```

## 字符串
### 字符串长度
```sh
s="12345"

echo ${#s}
```

### 截取字符串
```sh
s="012345abcde"

## 2345abcde
echo ${s:2}
```
1. 从字符串第2个开始截取一直到最末尾。
2. 字符串序列从0开始计数

```sh
s="012345abcde"

## abcd
echo ${s:6:4}
```
1. 从字符串第6个开始截取4个

### 删除字符串中的字符
#### 删除字符串右边的
```sh
file="sample.log.txt"

## 输出为sample.log
echo ${file%.*}
```
1. 从右往左匹配 
2. 删除%右边的通配符`.*`后面的字符  
3. 一个% 是非贪婪的，只匹配一个

```sh
file="sample.log.txt"

## 输出为sample
echo ${file%%.*}
```
1. 两个`%`是贪婪算法，匹配所有的

#### 删除字符串左边的
```sh
file="sample.log.txt"

## 输出为log.txt
echo ${file#*.}
```
1. 从左往右匹配 
2. 删除通配符`*.`左边的字符  
3. 一个#是非贪婪的，只匹配一个

```sh
file="sample.log.txt"

## 输出为txt
echo ${file##*.}
```
1. 两个`#`是贪婪算法，匹配所有的

### 字符串替换
```sh
s="1;2;3;4;5"

## 1_2;3;4;5
echo ${s/;/_}

## 1_2_3_4_5
echo ${s//;/_}

## 匹配前缀 _3;4;5
echo ${s/#1;2;/_}

## 匹配后缀 1;2;3;_
echo ${s/%4;5/_}
```
1. 只有一个 `/` 非贪婪替换
2. 两个 `/` 贪婪替换

### 字符串转化为数组
```sh
str="ONE,TWO,THREE,FOUR"
arr=(${str//,/ })
echo ${arr[@]}
```

### 字符串是否为空
```sh
[[ -z ${s} ]]
```
> True if the length of string is zero
> 字符串长度是0 则为真

### 字符串是否为非空
```sh
[[ -n ${s} ]]
```
> True if the length of string is non-zero
> 字符串长度是非0 则为真

### 设置字符串默认值
```sh
libva_rev=${LIBVA_REV:-"va-unified"}
```

这里的`:-` 把 `LIBVA_REV` 和 `"va-unified"` 分成左右2部分
有赋值 就剪掉 右边的
如果LIBVA_REV之前有赋值， 则等于`:-`左边 LIBVA_REV的值
如果LIBVA_REV没有赋值， 就等于`:-`右边 `LIBVA_REV="va-unified"`

```sh
pkg_config_path=${PKG_CONFIG_PATH:+":"$PKG_CONFIG_PATH}
```
这里的 `:+` 把 `PKG_CONFIG_PATH` 和 `":"$PKG_CONFIG_PATH` 分成2部分
有赋值 就加上 右边的
如果PKG_CONFIG_PATH之前有赋值， 则等于`:-`右边 `":"$PKG_CONFIG_PATH`
如果PKG_CONFIG_PATH没有赋值， 则等于`:-`左边也就是空

> 其实不用冒号:也成立

### 冒号
参考[csdn](https://blog.csdn.net/honghuzhilangzixin/article/details/7073312)

1. 啥也不做，只起到占位符的作用
```sh
if [ "${a}" == "1" ]; then
    :
else
    :
fi
```

2. 注释

单行注释
```sh
# your comment here
: your comment here
```

> #后的注释内容不会被检查，但:后的注释内容会被检查，如果有语句如果出现语法错误，则会报错 参考[这里](https://ixyzero.com/blog/archives/2512.html)

多行注释
```sh
: 'comment line1
comment line2
comment line3'
```

3. 清空文件的内容
```sh
:> a.txt

cat /dev/null > a.txt
```

4. 设置默认值
```sh
: ${COMPONENT_PROJECT:="123"}
COMPONENT_BRANCH=${COMPONENT_BRANCH:-"456"}

echo "[${COMPONENT_PROJECT}]"
echo "[${COMPONENT_BRANCH}]"
```

## 时间间隔
```sh
echo -n Count:

## 保存当前光标位置
tput sc

count=0;
while true; do
    if [ $count -lt 40 ]; then
        let count++;
        sleep 1;

        ## 显示输出 
        tput rc

        ## 删除当前光标位置到行尾的所有内容
        tput ed
        echo -n $count;
    else
        exit 0;
    fi
done
```

## 文件操作
### 烧制iso文件到DVD
```bash
sudo cdrecord -v -eject dev=/dev/cdwriter driveropts=burnfree CentOS.iso
```

### 得到执行脚本的路径
```bash
filepath=$(cd "$(dirname "$0")"; pwd)
```

### 删除以-开头的文件
```bash
touch -- -f
rm ./-f
```

### 按列合并文件
以 `|` 为分隔符， 合并
```sh
paste -d\| number.txt value.txt
```

将整列 以整行输出
```sh
paste -s value.txt
```

### cp 备份文件
```bash
cp some_file_name{,.bak}
```

### cp 拷贝隐藏文件
```sh
cp -a abc.git/. xyz.git/
```

> `cp -a abc.git/* xyz.git/`:
> abc.git下的隐藏文件都不会被拷贝，子目录下的隐藏文件倒是会的
>
> `cp -a abc.git/. xyz.git/`:
> 这样会把abc.git下的隐藏文件都会被拷过去

### 文件一行一行读取
```bash
cat abc.txt | while read line; do
    echo "[$line]"
done

while read line; do
    echo "[$line]"
done < abc.txt
```

### 多行文本
```sh
echo "<HTML>
    <HEAD>
          <TITLE>Page Title</TITLE>
    </HEAD>
    <BODY>
          Page body.
    </BODY>
</HTML>"
```

```sh
FOLDERS=`cat << EOF
/mnt/WEEKLY-BUILD/STAGING/SMTA/CENTOS
/mnt/WEEKLY-BUILD/STAGING/ASTA/CENTOS
/mnt/WEEKLY-BUILD/STAGING/SNVA/CENTOS
/mnt/WEEKLY-BUILD/STAGING/VCSA/CENTOS
/mnt/WEEKLY-BUILD/STAGING/LIBOA/CENTOS
/mnt/WEEKLY-BUILD/STAGING/Xcoder/CENTOS
EOF`
```

```sh
## write
(
cat << EOF
111111111111111
222222222222222
333333333333333
EOF
) > abc.txt
```

```sh
## 重定向
cat << EOF > xyz.txt
111111111111111
222222222222222
333333333333333
EOF

## 打印并重定向
cat << EOF | tee abc.txt
111111111111111
222222222222222
333333333333333
EOF

## 追加
cat << EOF >> xyz.txt
111111111111111
222222222222222
333333333333333
EOF
```

转义与不转义

EOF 转义
```sh
cat << EOF
$PATH
EOF
```

"EOF" 原样输出，不会转义
```sh
cat << "EOF"
$PATH
EOF
```


```sh
cat > tab.py <<EOF
#!/usr/local/bin/python
# python tab file
import sys
import readline
import rlcompleter
import atexit
import os
# tab completion
readline.parse_and_bind('tab: complete')
# history file
histfile = os.path.join(os.environ['HOME'], '.pythonhistory')
try:
    readline.read_history_file(histfile)
except IOError:
    pass
atexit.register(readline.write_history_file, histfile)

del os, histfile, readline, rlcompleter
EOF
```

### 多行注释
```sh
COMMENT_BLOCK=
if [ $COMMENT_BLOCK ]; then
echo "comment2"
echo "comment3"
fi

<<"COMMENT"
echo "command1"
echo "command2"
echo "command3"
COMMENT
```

### 将stdout和stderr 输出并重定向到文件
```sh
err.sh 2>&1 | tee -a log
```

### log 重定向
```bash
#!/bin/bash -exu

exec 3>&1 4>&2

trap 'exec 2>&4 1>&3' 0 1 2 3
exec 1> script.log 2>&1

pwd

ls 12345

date
```

### diff
生成 patch
```sh
diff -u version1 version2 > version.patch
```

打patch
```sh
patch -p1 version1 < version.patch
```

撤销已打的patch
```sh
patch -p1 -R version1 < version.patch
```

两个目录的diff
```sh
diff -Nuar dir1 dir2
```
1. N: 将所有缺失文件视为空文件
2. u: 生成一体化(unified)输出
3. a: 所有文件视为文本文件
4. r: 遍历所有文件

### shell里面help帮助文档
第一种: 使用here documents的方式写help信息

```sh
Help=$(cat <<-"HELP"

 my-script — does one thing well

 Usage:
   my-script <input> <output>

 Options:
   <input>   Input file to read.
   <output>  Output file to write. Use '-' for stdout.
   -h        Show this message.

HELP
)

help() {
    echo "$Help"
}

if [[ $# == 0 ]] || [[ "$1" == "-h" ]]; then
    help
    exit 1
fi

echo Hello World
```

第二种：help信息写在[注释里](https://gist.github.com/kovetskiy/a4bb510595b3a6b17bfd1bd9ac8bb4a5)

```sh
#!/bin/bash
###
### my-script — does one thing well
###
### Usage:
###   my-script <input> <output>
###
### Options:
###   <input>   Input file to read.
###   <output>  Output file to write. Use '-' for stdout.
###   -h        Show this message.

help() {
    sed -rn 's/^### ?//;T;p' "$0"
}

# if [[ $# == 0 ]] || [[ "$1" == "-h" ]]; then
if [[ $# == 0 || "$1" == "-h" ]]; then
    help
    exit 1
fi

echo Hello World
```

### shell 脚本接收重定向的内容
```sh
args=$(cat)
echo $args

cmd=$1
for i in $args; do
    $cmd $i
done
```

### 分割文本
将 passwd 这个文件 分割成 passwd-00, passwd-01 ...
每个文件1k 后缀是数字
```sh
split -b 1k passwd -d passwd-
```

将文件以正则分割为 read-00.log, read-01.log,  read-02.log
```sh
csplit README.md '/^# /' -n 2 {*} -f read- -b "%02d.log"
```
`'/^# /'`: 正则表达 开头为#
`-n 2`: 指定分割后文件名后缀数字个数
`-f read-`: 指定分割后文件名前缀
`{*}`: 根据匹配重复执行分割，直到文件末尾。 `{整数}`指定次数
`-b "%02d.log"`: 后缀格式

### 合并文本
```sh
cat a b c
```
a, b, c 依次相连

```sh
paste -d',' a b c
```
a 在第一列， b在第二列， c在第三列 每列以','分割

### 排序
```sh
$ cat alive
10.67.116.1 is alive
10.67.116.3 is alive
10.67.116.2 is alive
10.67.116.23 is alive
10.67.116.11 is alive

sort -t'.' -k 4 -n alive

sort -t '.' -k 3,3 -k 4,4 -n alive-2
```
`-t`: 分隔符
`-k 4`: 以哪一列排序
`-k 3,3`: 起始列3 结尾列3
`-k 3,3 -k 4,4`：先第3列，后第4列, `-k 3,4` 把3，4列作为一个整体
`-n`: 按数字number排序

## 通配符 globbing
shell [通配符][^globbing]通常用来**匹配文件或目录 而不是文本**

|      字符       |                              解释                              |
| :-------------- | :------------------------------------------------------------- |
| *               | 匹配任意长度任意字符                                           |
| ?               | 匹配任意单个字符                                               |
| [list]          | 匹配指定范围内（list）任意单个字符，也可以是单个字符组成的集合 |
| [^list]         | 匹配指定范围外的任意单个字符或字符集合                         |
| [!list]         | 同[^list]                                                      |
| {str1,str2,...} | 匹配 srt1 或者 srt2 或者更多字符串，也可以是集合               |


### 专用字符集
|    字符    |        意义        |
| :--------- | :----------------- |
| [:alnum:]  | 任意数字或者字母   |
| [:alpha:]  | 任意字母           |
| [:space:]  | 空格               |
| [:lower:]  | 小写字母           |
| [:digit:]  | 任意数字           |
| [:upper:]  | 任意大写字母       |
| [:cntrl:]  | 控制符             |
| [:graph:]  | 图形               |
| [:print:]  | 可打印字符         |
| [:punct:]  | 标点符号           |
| [:xdigit:] | 十六进制数         |
| [:blank:]  | 空白字符（未验证） |


> **Note:**
> 1. 在使用专属字符集的时候，字符集之外还需要用 [ ] 来包含住，否则专用字符集不会生效，例如 `[[:space:]]`
> 2. 想要转义的时候，单引号与双引号使用方法是不同的，单引号会转义所有字符，而且单引号中间不允许再出现单引号，双引号允许出现特定的 shell 元字符，具体字符可以自行查询
> 3. 在使用花括号 {} 的时候，里面的单个字符串需要使用单引号或者双引号括住，否则就会视为多个的单个字符 `ls {"au",[bc]}* `



## 目录
### 列出当前目录
```sh
ls -d */
```

### pushd 和 popd
```sh
➜  shm dirs
/dev/shm ~/Downloads /tmp/ISO /tmp

➜  shm pushd /var/lib/docker
/var/lib/docker /dev/shm ~/Downloads /tmp/ISO /tmp

➜  docker dirs
/var/lib/docker /dev/shm ~/Downloads /tmp/ISO /tmp

➜  docker popd +3
/var/lib/docker ~/Downloads /tmp/ISO /tmp

➜  docker pushd +2
~/Downloads /tmp/ISO /tmp /var/lib/docker
➜  Downloads
```

1. pushd和popd以栈的方式存储
2. 路径编号从0开始, **在ZSH下从右往左， Bash下从左往右**

### 树状显示
只标记出匹配样式的文件
```sh
tree -P '*.repo'
```

只标记出匹配样式之外的文件
```sh
tree -I '*.repo'
```

显示文件大小
```sh
tree -h
```

### log
log的配置文件在 /etc/rsyslog.d/

```sh
logger message
```

日志会写入到 /var/log/syslog  或 /var/log/messages

## 数组
```sh
array=(1 2 3 4 5)
array2=(
/proc/lockdep
/proc/lockdep_chains
/proc/lockdep_stat
/proc/locks
/proc/lock_stats
)

## 遍历
for i in ${array[@]}; do
    echo $i
done

## 数组长度
len=${#array[*]}

## 通过下标遍历
for((i=0; i<$len; i++));do
    echo ${array[$i]}
done
```

数组作为参数
```sh
print_array() {
    local list=$1
    for i in ${list[*]}; do
        echo ${i}
    done
}

list=(A B C D)
## must needs ""
print_array "${list[*]}"
```

## if
### if 使用正则
```sh
if [[ ! "${num}" =~ ^[1-9]+$ ]]; then
    exit 2
fi
```

> **Note**: =~ 右边不能有引号

### if 里面多个判断
只有单中括号 [ ] 是兼容 posix shell 标准的，比如 Bourne shell（/bin/sh）, ash, dash 这些，而其他一切双方括号，双圆括号都不兼容 posix shell 标准，而是 bash 特有的语法扩展[^shell-posix]。


```sh
linux_distro="ubuntu"
if [ ${linux_distro} == "Ubuntu" -o ${linux_distro} == "Debian" ]; then
    echo "debian like"
else
    echo "no"
fi

if [[ ${linux_distro} == "Ubuntu" ]] || [[ ${linux_distro} == "Debian" ]]; then
    echo "debian like"
else
    echo "no"
fi

if ( echo ${linux_distro} | grep 'ubuntu' ) || ( echo ${linux_distro} | grep 'debian' ) ; then
    echo "debian like"
else
    echo "no"
fi
```

## 脚本参数选项
若此脚本为 a.sh
```sh
while getopts "a:b:c" opt; do
    echo "[${opt}]: ${OPTARG}"
done
```

```sh
$ ./a.sh -a 123 -b 345
[a]: 123
[b]: 345
```

`getopts` 与 `case` 组合使用
```sh
while getopts "a:b:c" opt; do
    case ${opt} in
        a)
            echo "a = ${OPTARG}"
            ;;
        b)
            echo "b = ${OPTARG}"
            ;;
        c)
            echo "c = true"
            ;;
        ?)
            echo "unknown arg ${OPTARG}"
            ;;
    esac
done
shift $(($OPTIND-1))
```

```sh
$ ./a.sh -a 1 -b 2 -c
a = 1
b = 2
c = true
```
> 字符后面带有冒号后面一定要加参数 ./cmd -a XXX
> 字符后面没有冒号 ./cmd -c

getopts识别出各个选项之后，就可以配合case进行操作。操作中，有两个"常量"，
一个是OPTARG，用来获取当前选项的值；另外一个就是OPTIND，表示当前选项在参数列表中的位移。


## 解压缩
压缩后 删除源文件
```sh
tar czvf abc.tar.gz abc --remove-files
```

不把目录压缩进去
```sh
tar czvf abc.tar.gz -C /home/usr1/abc .
```
上面的方法有时候会失效

```sh
tar  -czfP  /root/44tar.gz  /root/ss2
```
[tar: Removing leading `/' from member names](https://www.cnblogs.com/operationhome/p/9802554.html) 这里加 `P` 避免这个。

解压后 删除第一层目录
```sh
tar xzvf abc.tar.gz --strip-components 1
```

## grep
```sh
grep 'version' . -r --include *.build  --include *.c
```
在*.build 和 *.c 的文件中 查找包含“version”

```sh
grep 'version' . -r --exclude README.md
grep 'version' . -r --exclude-dir .git
```

1. 排除 README.md 文件 查找匹配
2. 排除 .git 目录

### 比较
统计file1中没有，file2中有的行
```sh
grep -vwf file1 file2
```

### 行数
```sh
grep -m2 'nologin' /etc/passwd
```

## sed
读音 [sɛd] said

### 删除指定行
删除第1, 3, 5 这几行
```sh
sed -i '1d;3d;5d' a.txt
```

### 查找2个匹配行之间
```sh
sed -n '/Subject:/,/^diff --git /'p a.aptch
```

### 在行首或行尾添加
refer to [here](https://www.cnblogs.com/aaronwxb/archive/2011/08/19/2145364.html)

```sh
sed 's/^/HEAD&/g' test.file

sed 's/$/&TAIL/g' test.file
```

### 在2个匹配行之间的行首或行尾添加
```bash
echo -e '11111\n22222\n33333\n44444\n55555' | sed '/333/,/555/ s/^/#&/g'

echo -e '11111\n22222\n33333\n44444\n55555' | sed '/333/,/555/ s/$/&#/g'
```

### 匹配符合里面的第一行
```bash
sed -i '1,/enabled=0/{s/enabled=0/enabled=1/}' /etc/yum.repos.d/almalinux-powertools.repo
```

### 添加多行
```bash
sed -i '/Service]/a \
Environment=HTTP_PROXY=http://proxy-example.com:1234\
Environment=HTTPS_PROXY=http://proxy-example.com:1234\
NO_PROXY=localhost,127.0.0.0/8,*.example.com\
' docker.service
```

### windows 文件格式转换到 Linux下
```sh
sed -e 's/^M//g' windos_file
```
> `^M` 是先按 CTRL+V 再按 CTRL+SHIFT+M

### n命令
![](images/LinuxTips/sed_normal.png  "正常模式")
![](images/LinuxTips/sed_n.png  "n命令")

使用`n` 命令 之后， 把下一行内容覆盖 pattern space

```sh
# 偶数行打印2遍
sed 'n;p' file

#奇数行
sed ‐n 'p;n' test.txt 

#偶数行
sed ‐n 'n;p' test.txt 
```

### N命令
![](images/LinuxTips/sed_normal2.png  "正常模式")
![](images/LinuxTips/sed_big_n.png  "N命令")

使用`N` 命令 之后， 停在当前行，操作的是当前行和下一行2行

```sh
# 两行用：连接
sed 'N;s/\n/:/g' file
```

### 总结 sed n/N 命令
`echo -e "1\n2\n3\n4" | sed -n 'N;s/\n/ /;p'`
1. sed先读入第一行到pattern space，
2. 然后执行N命令，将第二行追加进pattern space
3. 这时pattern space里面就是`1\n2`，
4. 然后执行`s/\n/ /`，将换行符替换成空格，
5. 最后打印。

`echo -e "1\n2\n3\n4" | sed -n 'n;s/\n/ /;p'`
1. sed先读入第一行到pattern space，
2. 然后执行n命令，用第二行覆盖pattern space
3. 这时pattern space里面就是2，
4. 然后执行`s/\n/ /`，
5. 因为pattern space里没有`\n`，所以不做任何替换，直接打印

用shell来比喻的话:
N是:  echo 下一行内容 >> 模式空间
n是:  echo 下一行内容 >  模式空间


### 其他
这部分大部分来自 [陈皓 CoolShell](https://coolshell.cn/articles/9104.html)

去掉html中的标签
```sh
sed 's/<[^>]*>//g' a.html
```

使用`&`来当做被匹配的变量
```sh
[media@s5n8c2 t2]$ cat my.txt
This is my cat, my cat's name is betty
This is my dog, my dog's name is frank
This is my fish, my fish's name is george

[media@s5n8c2 t2]$ cat my.txt | sed 's/my/[&]/g'
This is [my] cat, [my] cat's name is betty
This is [my] dog, [my] dog's name is frank
This is [my] fish, [my] fish's name is george
```

使用1，2，3这些，可以只替换每一行的第1，2，3列
```sh
sed 's/s/S/2' my.txt
```

多个模式替换
```sh
sed '1,3s/my/your/g; 3,$s/This/That/g' my.txt
```

匹配 `\(`, `\)` 之内的可以当做变量来使用 `\1`, `\2`
```sh
➜ echo "AAA 123" | sed 's/\([A-Z]\+\) \([0-9]\+\)/\2 \1/'
123 AAA
```

```sh
[media@s5n8c2 t2]$ sed 's/This is my \([^,]*\),.*is \(.*\)/\1:\2/g' my.txt
cat:betty
dog:frank
fish:george
goat:adam
```

正则为：`This is my ([^,]*),.*is (.*)`  
匹配为：`This is my (cat),……….is (betty)`  
然后：`\1`就是cat，`\2`就是betty


## awk
读音 och [ɑk]

计算第一列的平均数
```sh
awk 'BEGIN{s=0} {s+=$1} END{print s/NR}'
```

根据文件中 包含def行，把文件分拆多份 
```sh
awk '/def/{n++} {print > "page"n".txt"}' gen_run_script.py
```

条件判断
```sh
objdump -tT libtest.so  | awk '{if($3=="DF" && $4==".text"){print $7}}'
```

获取最后一列，倒数第二列
```sh
awk -F":" '{print $2,$(NF),$(NF-1),$(NF-3)}' /etc/passwd
```

使用字典
```sh
history  | awk '{CMD[$2]++;count++} END{for (a in CMD) print a " == " CMD[a]/count*100 "%"}' | grep -v './' | column -t | sort -k3 -rn | cat -n |
head
```

awk从第二列到最后一列输出
```bash
ps | awk '{for (i=2;i<=NF;i++)printf("%s ", $i);print ""}'
```

## vim
### 保存时获得sudo权限
```
:w !sudo tee %
```

`%`: 当前编辑的文件名
`!cmd`: 执行shell命令
`:w !sudo tee %`: 通过管道把vim当前的缓存写入到当前文件中

### 每行行首，行尾操作
```
## 在每行行首添加
:%s/^/your_word/

## 在每行行尾添加
:%s/$/your_word/

## 在第2到第5行之间 行首添加 "#"
:2,5 s/^/#/

## 删除每行行尾多余空格
:%s/\s\+$//g

## 删除空行
:g/^$/d
```

### 格式化代码
```
:VG=
```

### 16进制
vim -b xxx

或者
```
:%!xxd
```

还原
```
:%!xxd -r
```

### 全局命令 `:g`

`:[range]g/pattern/command`

`:g!/pattern/d` 删除不匹配行
`:v/pattern/d` 删除不匹配行  v是in(v)erse的缩写
`:g/^$/d` 删除空行

`:1,20g/^/pu _` 1到20行 每一行插入空行
`:g/^[0-9]\{5\}/m$` 匹配行 移动到文件末尾
`:g/^[0-9]\{5\}/t$` 匹配行 复制到文件末尾
`:g/^/m0` 翻转文件中的每一行

> `:g/^/m0` 一行行匹配，匹配第一行时将第一行 m0 放到文件顶部，第二行放到文件顶部，当跑完一遍之后整个文件的每一行就反转了。

### win文档转换成linux
1. `:set ff=unix`
2. `:%/^M//g`  这里的 `^M` 是 Ctrl+V, Ctrl+M
3. dos2unix 命令 `dos2unix winFile`
4. sed 命令 `sed -i 's/^M//g'` 这里的 `^M` 是 Ctrl+V, Ctrl+M


## Bash 捕获 Ctrl + C
```sh
trap 'onCtrlC' INT
function onCtrlC() {
    echo "Ctrl + C is captured"
    exit 0
}

while true; do
    echo "I'm working"
    sleep 1
done
```

## 如何自定义Bash补全
[参考这里](https://tuzz.tech/blog/how-bash-completion-works)
```sh
complete -W "red green blue yellow purple pink orange" color

color p <Tab> <Tab>
```

## find
### -print0
-print: 以换行符作为分隔符
-print0: 以`\0` 也就是 `NULL` 作为分隔符

```sh
➜ ls -1
new file1.txt
new file2.txt
new file3.txt

➜ find . -name '*.txt' -print
./new file3.txt
./new file2.txt
./new file1.txt

➜ find . -name '*.txt' -print0
./new file3.txt./new file2.txt./new file1.txt

➜ find . -name '*.txt' -print0 | xargs rm
xargs: WARNING: a NUL character occurred in the input.  
It cannot be passed through in the argument list.  
Did you mean to use the --null option?
rm: cannot remove ‘./new’: No such file or directory
rm: cannot remove ‘file3.txt’: No such file or directory
rm: cannot remove ‘file2.txt’: No such file or directory
rm: cannot remove ‘file1.txt’: No such file or directory

➜ find . -name '*.txt' -print0 | xargs -p -0 rm
rm ./new file3.txt ./new file2.txt ./new file1.txt ?...y
```

### 找到文件不显示前缀./
```sh
find . -type f -printf '%P\n'

find * -type f
```

### find 删除带空格的文件
```sh
find . -name '*.html' -print0 | xargs -0 file
```

### find 把带有txt的移动到指定目录
```sh
find -iname "*txt*" -exec mv -v {} /home/user \;
```

### find 删除当前目录下超过100天的文件
```sh
find . -maxdepth 1 -ctime 100 -exec rm -rf {} \;
```

### find 找到可执行文件 并strip
```sh
find . -perm /u+x -a -type f | xargs strip
```

### find 多条件查询
```sh
find . -maxdepth 1 \( -name '*.pdf' -o -name '*.txt' -o -name '*.TXT' \) -print0 | xargs -0 md5sum
```

### find 排除某些文件
```sh
find . -path .repo -prune -o -name '.git' -a -type d -print

find . -path ./.git -prune -o -type f -print
```

> 这里的 `-o` 是短路求值
> 排除 .repo 后面 `-print` 不能少

## xargs
xargs命令的作用，是将标准输入转为命令行参数
```sh
echo "one two three" | xargs mkdir
```

```sh
## -d 分隔符
## -n 每行最多有几列
➜ echo "a1;b2;c3;d4" | xargs -d';' -n2
a1 b2
c3 d4

➜ find . -name '*.tar.gz' | xargs -n1 tar xzvf
```

1. -p: 询问用户是否执行
2. -t: 不询问直接执行

```sh
## -I 必须指定替换字符
find . -name '*.txt' | xargs -I {} cp {} tmp

find . -name '*.txt' -exec cp {} tmp2 \;
```

## Shell数组
通过下标遍历
```sh
s="abc"
ID=(1278 1280 1281 1282)

len=${#ID[*]}
echo $len
for i in $(seq 0 $len); do
    touch ${s}${ID[$i]}
done
```

直接遍历
```sh
IDs=(a
b
c)

for i in ${IDs[@]}; do
    echo ${i}
done
```

往数组后面追加
```sh
ARGS=()
ARGS+=(-a)
ARGS+=(-b)
ARGS+=(-c 123)

echo ${ARG[@]}
```

## bash 加上ex参数
```sh
#!/bin/bash -ex

## 如果没有找到 由于-e参数 这步会报错
which git-lfs &> /dev/null
if [ "$?" -ne "0" ]; then
    echo "git-lfs exists"
else
    echo "git-lfs not found"
fi

## 如果没有找到 即使-e参数 但这步不会报错
if ( which git-lfs &> /dev/null ); then
    echo "git-lfs exists"
else
    echo "git-lfs not found"
fi
```

- e: 遇到错误就退出
- x: 每条命令都会打印出来，类似于debug模式

## ffplay 播放 YUV
```sh
ffplay -autoexit -f rawvideo -video_size 176x96 test_stream_176x96.yuv
```

## expect
ssh
```
#!/usr/bin/expect

set timeout 30
spawn ssh user@localhost
expect {
    "(yes/no)" {
        send "yes\r"; exp_continue
     }
    "*assword" {
        send "123456\r"
    }
}

spawn ls
expect eof

#interact
```

scp
```
#!/usr/bin/expect
# script name : testExpect.sh

set timeout 30
set host localhost
set username user
set password 123456
set src_file [lindex $argv 0]
set dest_file [lindex $argv 1]

spawn scp $src_file $username@$host:$dest_file
expect {
    "(yes/no)" {
        send "yes\n"
        expect "*assword" { send "$password\n"}
    }

    "*assword" {
        send "$password\n"
    }
}

expect "100%"
expect eof
```

ssh + expect
```sh
#!/bin/bash

USER="$1"
PASS="$2"
HOST='192.168.0.5'
/usr/bin/expect << EOF
set timeout 60
spawn git clone $USER@$HOST:/home/user/myrepo
expect {
    "(yes/no)" {
        send "yes\n"; exp_continue
     }
    "*assword" {
        send "$PASS\n"
    }
}
expect eof
EOF
cd myrepo/
git checkout master
```

如果没有`exp_continue`这个命令，匹配第一个关键字以后就会继续匹配第二关键字，但有了这个命令后，匹配第一个关键字以后，第二次匹配依然从第一个关键字开始。


expect下的组合键
```sh
#!/usr/bin/expect
spawn minicom
expect "Press CTRL-A Z for help"
send "\07"      # Ctrl + G
send "0\n"
interact
```

> 组合键参考 http://blog.csdn.net/iodoo/article/details/49175749

## 备份
```sh
rsync -avrz  --delete --progress media@10.239.128.63:/home/user/ .
```

1. a: archive 进行归档
2. v: verbose 冗长信息
3. r: recursive 递归
4. z: compress 压缩
5. `--delete`: delete extraneous files from dest dirs 在dest删除无关文件
6. `--progress/-h`: show progress during transfer

> **Note:**   
> 末尾有`/` 就把 `/home/user/` 里面的内容同步，dest端是没有user这个文件夹的，   
> 末尾没有`/`, 就把`/home/user` 这个文件夹也同步过去， dest端是有user文件夹的

## 阻塞下输入
main.sh
```sh
read a
echo "aaaa: ${a}"

read b
echo "bbbb: ${b}"
```

```sh
(sleep 2
echo 1
sleep 2
echo 5
) | ./main.sh
```

上面可以有多个输入，下面的只能有一次输入。

```sh
./install.sh <<< "y"
```

## 编译、运行时几个环境变量
`PKG_CONFIG_PATH`: 为了寻找 *.pc 文件。 通过`pkg-config --cflags --libs` 指定系统中安装库。
```sh
➜ $ export PKG_CONFIG_PATH=/usr/lib64/pkgconfig:${PKG_CONFIG_PATH}

➜ $ pkg-config --cflags --libs libva
-lva
```

这个查看头文件来自哪里
```sh
$ pkg-config --variable=includedir libdrm
/usr/local/include

$ pkg-config --variable=libdir libdrm
/usr/local/lib64
```

> 上面 variable 的值都来自来自libdrm.pc

`LD_LIBRARY_PATH`: 运行需要的lib库
```sh
➜ $ export LD_LIBRARY_PATH=/usr/lib64:${LD_LIBRARY_PATH} 
```

找C/C++的头文件
```sh
➜ $ export C_INCLUDE_PATH=/opt/example/include
➜ $ export CPLUS_INCLUDE_PATH=/opt/example/include
```

库搜索路径，此环境变量指明路径会在-L指定路径之后，系统默认路径之前被搜索。
```sh
➜ $ export LIBRARY_PATH=/opt/example/lib
```

链接库搜索目录
```sh
gcc --print-search-dirs
```

头文件搜索目录
```sh
gcc -v -E - < /dev/null 2>&1 | awk '/^#include/,/^End of search/' | grep '^ '.

## for c++
gcc -xc++ -E -v - < /dev/null
```

> 编译时可以通过 `-nostdinc++`, `-nostdinc` 选项屏蔽对内定目录搜索头文件

## 看lib的版本
```sh
pkg-config --modversion libdrm
```

> pkg-config又是如何找到所需的.pc文件呢？  
> 需要用到一个环境变量PKG_CONFIG_PATH。这环境变量写明.pc文件的路径，pkg-config命令会读取这个环境变量的内容，这样就知道pc文件了。  
> `export PKG_CONFIG_PATH=/usr/lib64/pkgconfig:${PKG_CONFIG_PATH}`

## 看 so 用了哪个gcc版本编译的
```sh
objdump --full-contents --section=.comment libmfxhw64.so
```

## 看 lib 是不是 debug版本
```bash
readelf -S libxxx.so | grep -i '\.debug'
```

## patchelf 修改 runtime
```bash
patchelf \
    --set-interpreter /opt/glibc-2.27/lib/ld-linux-x86-64.so.2 \
    --set-rpath /opt/glibc-2.27/lib/ \
    --add-needed /opt/glibc-2.27/lib/libOpenCL.so.1 dpcpp

readelf -a dpcpp
```

`--set-interpreter`: 设置动态库解析器, 只针对binary，库文件无效
`--set-rpath`: 设置 rpath

## 看glibc 哪个版本
```sh
➜ $ ldd --version
ldd (GNU libc) 2.17
Copyright (C) 2012 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
```

```sh
➜ $ sudo ldconfig -p | grep libc.so
        libc.so.6 (libc6,x86-64, OS ABI: Linux 2.6.32) => /lib64/libc.so.6

➜ $ sudo ldconfig -v | grep libc.so
        libc.so.6 -> libc-2.17.so
```

## 看lib库中有哪些函数
```sh
nm /usr/local/lib/lib360SCVP.so
```
1. T类：是在**库中定义的函数**，用T **text (code) section**表示，这是最常见的；
2. U类：是在**库中被调用**，但并没有在库中定义(表明需要其他库支持)，用U **undefined**表示；
3. W类：是所谓的“弱态”符号，它们虽然在库中被定义，但是可能被其他库中的同名符号覆盖，用W **weak symbol**表示。

## git
### git-proxy
```sh
# Ubuntu
sudo apt-get install socat

# CentOS
sudo yum install socat
```

/usr/bin/git-proxy
```
proxy=proxy-example.com
exec socat STDIO SOCKS4:$proxy:$1:$2
```

~/.gitconfig
```
[core]
        gitproxy = none for example.com
        gitproxy = git-proxy
[http]
        proxy = http://name:password@example.com:914
        sslverify = false
[https]
        proxy = http://name:password@example.com:914
```

> 不在gitconfig里设置 [http] 和 [https], 就在终端里设置 
> export http_proxy=http://name:password@example.com:914
> export https_proxy=http://name:password@example.com:914

### git 指定ssh key
#### 使用 [GIT_SSH_COMMAND](https://blog.csdn.net/SCHOLAR_II/article/details/72191042)
```sh
GIT_SSH_COMMAND="ssh -i id_rsa" git clone ssh://user@example.com/demo.git
```

> **注意**，`-i` 有时可以被您的配置文件覆盖，在这种情况下，您应该给SSH一个空配置文件

```sh
GIT_SSH_COMMAND="ssh -i id_rsa -F /dev/null" git clone ssh://user@example.com/demo.git
```

#### 使用 `sshCommand` 为每个repo设置环境变量
```sh
git config core.sshCommand "ssh -i ~/.ssh/id_rsa -F /dev/null"
```

#### 使用 [ssh-agent](https://wangdoc.com/ssh/key.html#ssh-agent-%E5%91%BD%E4%BB%A4%EF%BC%8Cssh-add-%E5%91%BD%E4%BB%A4)
```sh
## ssh-agent bash

eval `ssh-agent`
ssh-add id_rsa
git clone ssh://user@example.com/demo.git

## 列出所有key
ssh-add -l

## 删除key
ssh-add -d id_rsa

## 删除所有key
ssh-add -D

## 退出 ssh-agent
ssh-agent -k
```

#### 使用 GIT_SSH
git.sh

```sh
if [ $# -eq 0 ]; then
    echo "Wrapper script that can specify an ssh-key file with the Git command
Usage:
    $0 -i ssh-key-file git-command"
    exit 1
fi

# remove temporary file on exit
trap 'rm -f /tmp/.git_ssh.$$' 0

if [ "$1" = "-i" ]; then
    SSH_KEY=$2; shift; shift
    echo "ssh -i "$SSH_KEY" \"\$@\"" > /tmp/.git_ssh.$$
    chmod +x /tmp/.git_ssh.$$
    export GIT_SSH=/tmp/.git_ssh.$$
fi

# in case the git command is repeated
[ "$1" = "git" ] && shift

# Run the git command
git "$@"
```

```sh
./git.sh -i id_rsa.pub git clone git@$server_ip:xxx.git
```

## 终端
### 踢出已登录用户
```
[media@localhost WebStream]$ w
 08:46:02 up 1 day, 17:47,  3 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
media    :0       :0               Tue14   ?xdm?  14:44   0.63s gdm-session-worker [pam/gdm-autologin]
media    pts/0    shwdeisgchi123.c Tue15    2.00s  0.08s  0.00s tmux new -s zbq
media    pts/7    shwdeisgchi123.c 08:44    1:41   0.08s  0.08s -bash

[media@localhost WebStream]$ pkill -kill -t pts/7

[media@localhost WebStream]$ pkill -t pts/0
```

### 向其他终端发送消息
```sh
cat message | wall

echo -e "hello123\n456" > /dev/pts/12
```

- `mesy y` 允许写入， `mesy n` 不允许写入。 发现行不通。

## 网络
### ping
```sh
for i in 10.67.114.{1..255};do (
    ping ${i} -c2 &> /dev/null
    if [ "$?" = "0" ]; then
        echo $i
    fi )&
done
wait
```

1. `()&` 子shell放后台运行
2. `wait` 等待所有的子进程结束
3. `$!` 进程号

```sh
sleep 5 &
pid=$!
wait $pid
```


### wget 
下载整个目录
```sh
wget --no-check-certificate --user ${USER} --password ${MY_PASSWORD} -r -np -nd ${URL}
wget -c -r -np -k -L -p -nc ${URL}
```

-c：继续执行上次终端的任务
-L：仅顺着关联的连接
-r：递归下载方式
-nc：文件存在时，下载文件不覆盖原有文件
-np：不查询父目录
-nd: 表示不在本机重新创建目录结构
-p：下载页面所需所有资源，如图片、声音等
-k：将下载内容中的链接转换为本地连接

### 通过IP查询mac地址
arp查询
```sh
ping 10.67.116.92
arp -a
```

### nmap
查询整个网段哪些服务端口的状态
```sh
nmap 10.67.116.0/23
```

指定范围
```sh
nmap 10.67.116.90-110
```

nmap 查询 hostname --> IP
```sh
nmap -sP -n 10.67.116.92
```

### 通过 traceroute 查询走了哪些路由
```sh
[root@10a96d9528f6 WS]# traceroute 10.67.116.91
traceroute to 10.67.116.91 (10.67.116.91), 30 hops max, 60 byte packets
 1  kabrenes-dev-03.amr.corp.example.com (172.18.0.1)  0.089 ms  0.028 ms  0.021 ms
 2  shz82b-ldr01-v512-pri.sh.example.com (10.67.112.2)  0.521 ms  0.662 ms  0.793 ms
 3  media-clips2.sh.example.com (10.67.116.91)  0.200 ms  0.178 ms  0.155 ms
```

### 建立网桥
```sh
sudo apt-get install bridge-utils
sudo brctl addbr br1
sudo brctl addif br1 eth0
sudo ifconfig eth0 0.0.0.0
sudo ifconfig br1 up
```

### 删除网桥
```sh
sudo ifconfig br1 down
sudo delif br1 eth0
sudo delbr br1
```

### 查看连接到网桥上的 ip和mac
```sh
#sudo yum install -y arp-scan
#sudo apt install -y arp-scan

sudo arp-scan --localnet --interface docker0
sudo arp-scan -l -I docker0
```

### 添加默认网关
方法1
```sh
#route add -net 10.239.173.0/24 enp1s0f1
#route add -net 192.168.100.0/24 eth0
route add default gw 10.239.173.1

ip route add default via 10.239.173.1 dev eth0
```

方法2
/etc/sysconfig/network
```sh
GATEWAYDEV=enp1s0f1
GATEWAY=10.239.173.1
```

### 禁止ping通
临时修改
```sh
# 禁止ping
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

# 允许ping
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
```

永久修改
修改 /etc/sysctl.conf
```
net.ipv4.icmp_echo_ignore_all=1
```

生效
```sh
sudo sysctl -p
```

### DNS
当出现 resolv.conf 链接到 stub-resolv.conf
```sh
$ file /etc/resolv.conf
/etc/resolv.conf: symbolic link to ../run/systemd/resolve/stub-resolv.conf
```

可以改变它的路径
```sh
ln -sfv /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### hostname 与 IP
通过hostname查询IP
```sh
ping build1.example.com
dig build1.example.com
```

```sh
$ host www.163.com
www.163.com is an alias for www.163.com.163jiasu.com.
www.163.com.163jiasu.com is an alias for www.163.com.lxdns.com.
www.163.com.lxdns.com has address 218.1.70.80
www.163.com.lxdns.com has IPv6 address 240e:f3:a000:1::6e
```

```sh
$ nslookup www.163.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
www.163.com     canonical name = www.163.com.163jiasu.com.
www.163.com.163jiasu.com        canonical name = www.163.com.lxdns.com.
Name:   www.163.com.lxdns.com
Address: 218.1.70.80
Name:   www.163.com.lxdns.com
Address: 240e:f3:a000:1::6e
```

通过ip查询hostname
```sh
host 10.67.116.101
dig -x 10.67.116.96
```

### nmcli
一个连接(connection)就是/etc/sysconfig/network-scripts/目录下的一个配置文件，  
接口(device)是物理设备，一个物理设备可以拥有多个配置文件，  
但只能有一个配置文件属于使用(active)状态；配置文件的生成与使用状态均由NetworkManager控制。 参考于[这里](https://www.jianshu.com/p/5d5560e9e26a)

```sh
sudo nmcli device disconnect eth1
sudo nmcli device connect eth1
sudo nmcli dev list eth1

sudo nmcli con add type ethernet con-name dynamic-bi ifname eth1
sudo nmcli con down dynamic-bi
sudo nmcli con up dynamic-bi
sudo nmcli con show dynamic-bi

sudo nmcli con add type ethernet con-name static-bi ifname eth1
sudo nmcli con modify static-bi \
    ipv4.method static \
    ipv4.addresses 192.168.0.5/24 \
    ipv4.dns "10.248.2.5" \
    connection.autoconnect no
sudo nmcli con down static-bi
sudo nmcli con up static-bi

sudo nmcli connection delete dynamic-bi
```

当出现出现
```sh
$ nmcli device
DEVICE   TYPE      STATE      CONNECTION
docker0  bridge    unmanaged  --
ens10f0  ethernet  unmanaged  --
ens10f1  ethernet  unmanaged  --
```
参考[ethernet-device-not-managed](https://askubuntu.com/questions/882806/ethernet-device-not-managed)

在 /etc/NetworkManager/conf.d/ 或 /usr/lib/NetworkManager/conf.d/
找到 10-globally-managed-devices.conf
```ini
[keyfile]
unmanaged-devices=*,except:type:wifi,except:type:gsm,except:type:cdma
```

```sh
sudo mv 10-globally-managed-devices.conf 10-globally-managed-devices.conf.orig
```

重启服务
```sh
sudo systemctl restart NetworkManager
sudo nmcli device set ens10f1 managed yes
```

上面的方法可以未必生效 参考[这里](https://superuser.com/questions/1429490/unmanaged-network-manager-in-ubuntu)试试下面方法

在 /etc/NetworkManager/conf.d/10-globally-managed-devices.conf 中
```sh
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=true

[keyfile]
unmanaged-devices=*,except:type:wifi,except:type:gsm,except:type:cdma,except:type:wwan,except:type:ethernet,except:type:vlan

[device]
wifi.scan-rand-mac-address=no
```

重启服务
```sh
sudo systemctl restart NetworkManager
sudo nmcli device set ens10f1 managed yes
```

不知道为啥会得到2个动态IP, 参考[这里](https://askubuntu.com/questions/1030957/second-ip-via-dhcp-why)
```sh
sudo ip addr flush dev eno1
```

### ip 命令
参看[这里](http://blog.csdn.net/superbfly/article/details/49467159)

#### 开启/关闭网卡
相当于 `ifconfig eth0 up/down`
```sh
ip link set dev eth0 up/down
```

#### 修改MAC地址
```sh
ip link set dev eth0 address 00:01:4f:00:15:f1
```

#### 查看ip地址
```sh
ip addr show
```

#### 查看路由表
```sh
ip route show
```

#### 网关
添加默认网关
```sh
ip route add default via 192.168.137.2
```

删除默认网关
```sh
ip route del default
```

### 端口
```sh
cat /etc/services
```

参看远程某个端口是否打开
```sh
nmap ${IP} -p ${PORT}

## 远程端口是否打开 z:zero-I/O
#yum install -y nmap-ncat
#apt install -y nc
nc -zv ${IP} ${PORT}

telnet ${IP} ${PORT}
```

查看ipv4网络连接
```sh
lsof -i 4
```

查看端口
```sh
lsof -i:22
```

```sh
netstat -antp
```
1. a: 显示所有监听以及不监听的
2. n: ip，而不是hostname
3. t: tcp
4. p: 显示program的PID和name

参看本地哪些端口已经打开
```sh
nmap --reason 127.0.0.1
```

### bonding
```sh
[media@localhost ~]$ uuidgen enp4s0f0
6d226496-1621-4ba2-8c81-baefccb24b60
[media@localhost ~]$ uuidgen enp4s0f1
535e015e-cd2e-4a1b-9b28-85ca03f7cefa
```

/etc/sysconfig/network-scripts/ifcfg-bond0
```
TYPE=Bond   #Interface type set to bond
BOOTPROTO=static
BONDING_MASTER=yes
BONDING_OPTS="mode=active-backup"  #i set mode to active-backup 
DEFROUTE=yes 
IPADDR=192.168.122.100  #IP address of bond0 interface
NETMASK=255.255.255.0 
GATEWAY=192.168.122.1
IPV4_FAILURE_FATAL=no 
IPV6INIT=no
NAME=bond0
DEVICE=bond0
ONBOOT=yes
```

/etc/sysconfig/network-scripts/ifcfg-enp4s0f0
```
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
NAME=enp4s0f0
UUID=6d226496-1621-4ba2-8c81-baefccb24b60
DEVICE=enp4s0f0
ONBOOT=yes
```

/etc/sysconfig/network-scripts/ifcfg-enp4s0f1
```
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
NAME=enp4s0f1
UUID=d46459b8-4500-9a0b-79f0c65ca109
DEVICE=enp4s0f1
ONBOOT=yes
```

### teaming
/etc/sysconfig/network-scripts/ifcfg-team0
```
#BOOTPROTO=static
BOOTPROTO=dhcp
TEAM_CONFIG='{"runner":{"name":"activebackup"},"link_watch":{"name":"ethtool"}}'
NAME=team0
DEVICE=team0
ONBOOT=yes
DEVICETYPE=Team
#IPADDR=192.168.122.120
#NETMASK=255.255.255.0
#GATEWAY=192.168.122.1
#PREFIX=24
```

/etc/sysconfig/network-scripts/ifcfg-enp4s0f0
```
TEAM_MASTER=team0
TEAM_PORT_CONFIG='{"prio":99}'
DEVICETYPE=TeamPort
NAME=enp4s0f0
HWADDR=90:e2:ba:4a:33:78
DEVICE=enp4s0f0
ONBOOT=yes
```

/etc/sysconfig/network-scripts/ifcfg-enp4s0f1
```
TEAM_MASTER=team0
TEAM_PORT_CONFIG='{"prio":100}'
DEVICETYPE=TeamPort
NAME=enp4s0f1
HWADDR=90:e2:ba:4a:33:79
DEVICE=enp4s0f1
ONBOOT=yes
```


## LVM
local volume manager

![](images/LinuxTips/LVM.png  "LVM")

上图中在HDD层需要把/dev/sda1 这些在分区时系统ID, fdisk改为8e, gdisk改为8e00 也就是Linux LVM类型。

上图中PE就是Physical Extent。类似于 ext3 里面block的概念。

命令表
|   任务    |    PV     |        VG         |     LV     |      FS       |
| :-------- | :-------- | :---------------- | :--------- | :------------ |
| scan      | pvscan    | vgscan            | lvscan     | lsblk,blkid   |
| create    | pvcreate  | vgcreate          | lvcreate   | mkfs.ext4     |
| display   | pvdisplay | vgdisplay         | lvdisplay  | df, mount     |
| Display   | pvs       | vgs               | lvs        |               |
| extend    | vgextend  | vgextend/lvresize | xfs_growfs | resize2fs     |
| reduce    | vgreduce  | lvreduce/lvresize | None       | resize2fs     |
| remove    | pvremove  | vgremove          | lvremove   | umount,格式化 |
| resize    | lvresize  | xfs_growfs        | resize2fs  |               |
| attribute | pvchange  | vgchange          | lvchange   | /etc/fstab    |


### 创建 PV
```sh
sudo pvcreate /dev/sdb1
sudo pvdisplay
```

### 删除 PV
```sh
sudo pvremove /dev/sdb1
```

### 创建 VG
```sh
sudo vgcreate volume-group1 /dev/sdb1 /dev/sdb2
sudo vgdisplay
```

### 删除 VG
```sh
sudo vgremove volume-group1
```

### 扩展 VG
```sh
sudo vgextend volume-group1 /dev/sdb3
```

### 创建 LV
```sh
sudo lvcreate -L 100M -n lv1 volume-group1
sudo lvdisplay
```

为其格式化为ext4格式
```sh
sudo mkfs.ext4 /dev/volume-group1/lv1
mkdir lv1-mount
sudo mount /dev/volume-group1/lv1 lv1-mount
```

### 修改 LV 的大小
```sh
sudo lvresize -L 200M /dev/volume-group1/lv1

## 增加PE大小
sudo lvresize -l +884 /dev/volume-group1/lv1
```

### 检查磁盘错误
```sh
sudo e2fsck /dev/volume-group1/lv1
sudo e2fsck -f /dev/volume-group1/lv1
```

### 更新文件系统
```sh
sudo resize2fs /dev/volume-group1/lv1
sudo lvdisplay
```

上面有时候报错，下面的却可以
```sh
sudo xfs_growfs -d /
```

### 挂载 LVM
```sh
sudo lvdisplay

--- Logical volume ---
LV Path                /dev/centos_lh/home
LV Name                home
VG Name                centos_lh
LV UUID                1qiRhY-AVX4-Wueh-mUp5-mER0-ISCM-E0wwlp
LV Write Access        read/write
LV Creation host, time lh.lm, 2016-08-30 10:58:18 +0800
LV Status              available
# open                 1
LV Size                873.21 GiB
Current LE             223541
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
Block device           253:2
```

```sh
## 设为活动
sudo lvchange -ay /dev/centos_lh/home
sudo mount /dev/centos_lh/home /mnt
```

### 重命名同名的VG
```sh
$ sudo pvs
  PV         VG     Fmt  Attr PSize    PFree
  /dev/sda2  centos lvm2 a--  <465.27g    0
  /dev/sdb2  centos lvm2 a--  <931.02g    0

$ sudo vgs -v
    Archiving volume group "centos" metadata (seqno 4).
    Archiving volume group "centos" metadata (seqno 4).
    Creating volume group backup "/etc/lvm/backup/centos" (seqno 4).
    Cache: Duplicate VG name centos: Prefer existing 7fo5pW-bjKx-2wzi-kJez-2nOj-WuuD-Mxk3vy vs new PzOElX-4jRk-JCL4-8dJx-t7c0-7xOl-oeyjwM
    Archiving volume group "centos" metadata (seqno 4).
    Archiving volume group "centos" metadata (seqno 4).
    Creating volume group backup "/etc/lvm/backup/centos" (seqno 4).
  VG     Attr   Ext   #PV #LV #SN VSize    VFree VG UUID                                VProfile
  centos wz--n- 4.00m   1   3   0 <465.27g    0  7fo5pW-bjKx-2wzi-kJez-2nOj-WuuD-Mxk3vy
  centos wz--n- 4.00m   1   3   0 <931.02g    0  PzOElX-4jRk-JCL4-8dJx-t7c0-7xOl-oeyjwM

$ sudo vgrename PzOElX-4jRk-JCL4-8dJx-t7c0-7xOl-oeyjwM centos1t
  Processing VG centos because of matching UUID PzOElX-4jRk-JCL4-8dJx-t7c0-7xOl-oeyjwM
  Volume group "PzOElX-4jRk-JCL4-8dJx-t7c0-7xOl-oeyjwM" successfully renamed to "centos1t"

$ sudo pvs
  PV         VG       Fmt  Attr PSize    PFree
  /dev/sda2  centos   lvm2 a--  <465.27g    0
  /dev/sdb2  centos1t lvm2 a--  <931.02g    0

$ sudo lvscan
  ACTIVE            '/dev/centos/home' [<407.63 GiB] inherit
  ACTIVE            '/dev/centos/swap' [7.64 GiB] inherit
  ACTIVE            '/dev/centos/root' [50.00 GiB] inherit
  inactive          '/dev/centos1t/home' [<873.38 GiB] inherit
  inactive          '/dev/centos1t/swap' [7.64 GiB] inherit
  inactive          '/dev/centos1t/root' [50.00 GiB] inherit

$ sudo vgchange -ay /dev/centos1t
  3 logical volume(s) in volume group "centos1t" now active

$ sudo lvscan
  ACTIVE            '/dev/centos/home' [<407.63 GiB] inherit
  ACTIVE            '/dev/centos/swap' [7.64 GiB] inherit
  ACTIVE            '/dev/centos/root' [50.00 GiB] inherit
  ACTIVE            '/dev/centos1t/home' [<873.38 GiB] inherit
  ACTIVE            '/dev/centos1t/swap' [7.64 GiB] inherit
  ACTIVE            '/dev/centos1t/root' [50.00 GiB] inherit

$ sudo mount /dev/centos1t/home /mnt/
```

## 磁盘
### 查看磁盘信息
这个有些查不到
```sh
# apt-get install -y smartmontools
# yum install -y smartmontools

sudo smartctl -a /dev/sda
```

这个是系统自带的
```sh
udevadm info  --query=all  --name=/dev/sda | grep -i serial
```


### 大硬盘格式化及分区
用parted分区
```sh
sudo parted /dev/sdb
mklabel gpt 
print
mkpart primary 0 8TB
quit
```

用gdisk分区
```sh
sudo gdisk /dev/sdb
```

```sh
sudo mkfs.ext4 -T largefile /dev/sdb1
```

### xfs文件系统修复
```sh
ls -l /dev/mapper
xfs_repair /dev/mapper/cl_muban-root
xfs_repair -L /dev/mapper/cl_muban-root
```

> `-L` 强制 日志 归零

### 查找磁盘最大的文件
```sh
du -a | sort -rnk 1 | head

find . -type f -exec du -s {} \; | sort -rnk 1 | head
```

## 清理缓存
```sh
echo 3 > /proc/sys/vm/drop_caches
```

## 时间 时区
### 时间显示用英文
临时生效
```sh
export LC_TIME="en_US.UTF-8"
```

永久生效 修改 `/etc/default/locale`
```
LANG="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
```

或者
```sh
sudo localectl set-locale LANG=en_US.utf8
```

### 修改日期 和 时间
```sh
## 手动设置
sudo date -s "2016/08/17 14:37:00"

## 与NTP服务器同步
sudo ntpdate corp.example.com

## 系统时间写入BIOS
sudo hwclock -w

## 从BIOS时间写入系统
sudo hwclock -s
```

### 修改时区
```sh
## 列出所有时区
sudo timedatectl list-timezones

## 修改时区
sudo timedatectl set-timezone Asia/Shanghai
```

或者用创建软连接的方式
```sh
sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 在Linux访问windows
```sh
rdesktop -u USER -p PASSWD IP
```
>  Windows下 要先开启远程桌面访问权限

## CPU
### CPU 信息
物理CPU个数
```sh
grep "physical id" /proc/cpuinfo | sort | uniq | wc -l
```

每个物理CPU的核数
```
grep "cpu cores" /proc/cpuinfo | uniq
```

线程个数
```sh
grep "processor" /proc/cpuinfo | wc -l
```

是否启动超线程
```sh
cat /proc/cpuinfo | grep -e "cpu cores"  -e "siblings" | sort | uniq
```

> 如果 `cpu cores` 和 `siblings` 的数量一致就没有启动了超线程， 否则就启动了超线程

另一种从`lscpu`来看
```sh
$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
CPU(s):              112
Thread(s) per core:  2
Core(s) per socket:  28
Socket(s):           2
```
Sockert：有多少个CPU插槽
Core(s) per socket: 每一个插槽上面的物理CPU有多少个核
Thread(s) per core: 每个核上面有多少线程


CPU型号
```sh
grep name /proc/cpuinfo | uniq
```

### CPU 锁频
锁住第i个CPU为2MHz
```sh
echo 2000000 > /sys/devices/system/cpu/cpu${i}/cpufreq/scaling_min_freq
echo 2000000 > /sys/devices/system/cpu/cpu${i}/cpufreq/scaling_max_freq
cat /proc/cpuinfo | grep 'cpu MHz'
```

### 限定CPU个数
```sh
echo 1 > /sys/devices/system/cpu/cpu${i}/online

echo 0 > /sys/devices/system/cpu/cpu${i}/online
```
1: 表示启动
0：表示下线

> **注意：** cpu0这个是主CPU，不让修改，没有cpu0/online

或者修改开机启动项
```
maxcpus=4 nr_cpus=4
```
maxcpus=4: 最多使用4个CPU
nr_cpus=4: 当前运行4个CPU

## GPU
### 集显
锁频GPU 700MHz
```sh
echo 700 > /sys/kernel/debug/dri/0/i915_max_freq
echo 700 > /sys/kernel/debug/dri/0/i915_min_freq
```

### SG1 锁频
```sh
lspci | grep -i vga | grep "Intel Corporation Device"
1a:00.0 VGA compatible controller: Intel Corporation Device 4905
```

这里的 **1a**:00.0
找到  /sys/bus/pci/devices/0000:**1a**:00.0/drm/card0

锁频 1000 Mhz
```sh
echo 1000 > /sys/bus/pci/devices/0000:1a:00.0/drm/card0/gt_boost_freq_mhz
echo 1000 > /sys/bus/pci/devices/0000:1a:00.0/drm/card0/gt_max_freq_mhz
echo 1000 > /sys/bus/pci/devices/0000:1a:00.0/drm/card0/gt_min_freq_mhz
```

## 内存
### 看内存物理信息
```sh
sudo dmidecode -t memory

sudo lshw -class memory
```

### 内存使用
```sh
cat /proc/meminfo
free
```

### 从频率看DDR几
DDR1: 333MHZ和400MHZ
DDR2: 533,667,800
DDR3: 1066,1333,1600,2133
DDR4: 2133,2400,3000,3333

时钟频率33.33 MHz的晶振是性能和成本比最便宜的晶振之一，
将33.33 MHz作为通用CPU时钟及PCI总线时钟的频率有很大一部分是此原因。

主板 BIOS 处会有一个叫 Ratio 的设置，可以设置成 100:100 或者 100:133, 目的就是转换成 133 MHz，让内存能和主板、CPU 能够异步工作。

![](images/LinuxTips/memory-ddr.png  "DDR")


## 进程
占用CPU最多的10个程序
```sh
ps -eo comm,pcpu,pmem --sort -pcpu | head
```

杀死进程
```sh
kill -9 1402

killall -9 gedit

pkill -s gedit
```

### 捕捉并响应信号
`trap`命令在脚本中为信号分配信号处理程序。

```sh
handle() {
    echo Hey, received signal: SIGINT
    echo My process ID is: $$
}

trap 'handle' SIGINT

while true; do
    sleep 2
done
```

### 杀死进程
```sh
killall ffplay

kill $(pidof ffplay)
```
这里的 `killall` 有个缺点，如果命令名太长可能匹配不到

# Bash 快捷键

| 操作                                                                                                                     | 快捷键  |
| :-----                                                                                                                   | :---    |
| 移动光标至行首                                                                                                           | CTRL+A  |
| 第一次按时，移动光标至行首再次按时，回到原有位置                                                                         | CTRL+X  |
| 移动光标至行尾                                                                                                           | CTRL+E  |
| 光标向左移动一个字符                                                                                                     | CTRL+B  |
| 光标向左移动一个单词                                                                                                     | ESC+B   |
| 光标向右移动一个字符                                                                                                     | CTRL+F  |
| 光标向右移动一个单词                                                                                                     | ESC+F   |
| 删除光标前一个字符，即退格（Backspace）                                                                                  | CTRL+H  |
| 删除光标前所有字符                                                                                                       | CTRL+U  |
| 删除光标后所有字符                                                                                                       | CTRL+K  |
| 删除光标前一个单词                                                                                                       | CTRL+W  |
| 删除光标后一个字符，（相当于Delete）无任何字符时相当于exit                                                               | CTRL+D  |
| 粘贴之前（CTRL+U/K/W）删除的内容                                                                                         | CTRL+Y  |
| 清屏，相当于指令 clear                                                                                                   | CTRL+L  |
| 查找并自动匹配之前使用过的指令                                                                                           | CTRL+R  |
| 回车，相当于Enter                                                                                                        | CTRL+M  |
| 跳到新行，等同于回车                                                                                                     | CTRL+O  |
| 新起一行，命令行下等同于回车                                                                                             | CTRL+J  |
| 横行制表符，在命令行中补齐指令， 效果和Tab键相同                                                                         | CTRL+ I |
| 补齐指令                                                                                                                 | TAB     |
| 上一条指令，等同于向上箭                                                                                                 | CTRL+P  |
| 下一条指令，等同于向下箭                                                                                                 | CTRL+N  |
| 中断操作                                                                                                                 | CTRL+C  |
| 冻结终端操作（暂停脚本）                                                                                                 | CTRL+S  |
| 恢复冻结（继续执行脚本）                                                                                                 | CTRL+Q  |
| 使下一个单词首字母大写, 同时光标前进一个单词, 如光标停留在单词的某个字母上, 如word中的o字母上, 则o字母变大写. 而不是w    | ESC+C   |
| 使下一个单词所有字母变大写, 同时光标前进一个单词； 如光标在o字母上, 则ord变大写, w不变.                                  | ESC+U   |
| 使下一个单词所有字母变小写, 同时光标前进一个单词； 如光标在o字母上, 则ord变小写, w不变.                                  | ESC+I   |
| 将光标处的字符和光标前一个字符替换位置                                                                                   | CTRL+T  |
| 重复运行最近一条以“word”开头的指令，如!ls 或 !l                                          | !word   |
| 调用上一条指令的最后一个参数作为当前指令对象, 如，假设上一条指令为： ls abc.txt bbc.txt 那么， vi !$ 相当于： vi bbc.txt | !$      |
| 调用执行指定编号的历史记录指令,如!2, !11                                                                                 | !number |

[^shell-posix]: https://www.zhihu.com/question/266787434 作者: 韦易笑
[^globbing]: https://www.cnblogs.com/divent/p/5762154.html 作者: divent

