# Linux 开机启动脚本

## 准备开机脚本
sendip

```bash
#!/bin/bash
#chkconfig: 2345 95 5
#description: script to send IP to someone

subject="IP address changed from last reboot"
sendto=your@mail.com
old_ip_file=/tmp/lan_ip_addrc
maildate_file=/tmp/maildata

send_ip_if_changed()
{
    host_name=`hostname`

    n=0
    timeout=600
    while true;do
        ip_addr=`ifconfig | grep "10.239" | awk '{ print $2 }' | cut -d":" -f2`
        if [ "${ip_addr}" != "" ]; then
            break
        fi
        n=$((n+1))
        sleep 1
        if [ "${n}" -gt "${timeout}" ]; then
            break
        fi
    done

    old_ip_addr=`cat ${old_ip_file}`
    if [ "${ip_addr}" != "${old_ip_addr}" ] ;then
        echo "Host ${host_name} changed IP to ${ip_addr}"
        echo "HostName=${host_name}" > ${maildate_file}
        echo "IPAddr=${ip_addr}" >> ${maildate_file}
        osinfo=`cat /etc/issue | sed '/^$/'d | head -1`
        #echo ${osinfo}
        echo "OS:${osinfo}" >> ${maildate_file}
        mail -s "${subject}" ${sendto} < ${maildate_file}
        rm -rf ${maildate_file}
        echo "${ip_addr}" > ${old_ip_file}
    fi
}

case $1 in
start)
    send_ip_if_changed
    ;;
stop)
    echo "Nothing to do here"
    ;;
restart)
    send_ip_if_changed
    ;;
*)
    echo "Usage: $0 [start|stop|restart]"
    exit 1
;;
esac

exit 0
```

## Ubuntu12.04
```bash
sudo apt-get install mailutils
sudo cp sendip /etc/init.d
cd /etc/init.d
sudo update-rc.d sendip defaults 95
```

> 在Debain系统下 要添加如下信息替换什么的注释

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides: sendip
# Required-Start:
# Should-Start:
# Required-Stop:
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: sendip
# Description: Start Sendip
### END INIT INFO
```

## CentOS6.5
```bash
sudo cp sendip /etc/init.d
cd /etc/init.d
sudo chkconfig --add sendip
sudo chkconfig --level 2345 sendip on
sudo chkconfig --list | grep 'sendip'
```

> 也可以写在开机脚本 /etc/rc.d/rc.local 中

___


## CentOS7
```bash
sudo systemctl status rc-local.serives     #查看服务
sudo systemctl enable rc-local.service     #启动服务
sudo cp sendip /etc/init.d/                
```


在开机启动脚本/etc/rc.local中 添加启动命令
```bash
cat /etc/rc.local
/etc/init.d/sendip start
```

---

## SUSE11sp3

[SUSE 参考这里](http://blog.csdn.net/rokii/article/details/6316443)

```bash
sudo cp sendip /etc/init.d
cd /etc/init.d

```

SUSE在 /etc/init.d 下的几个文件
1. boot.local –> 在 rc5.d 前就有动作
2. halt.local –> 这个关机启动档案会在最后有动作
3. before.local –> 这个档案比较用不到所以不需多做解释
4. after.local –> 这个档案会在 rc5.d 之后有动作 , 就是最重要的开机启动档 ， 没有的话 新建一个
 
上面第三及第四个档案预设是不存在的喔!!
当你看过 /etc/init.d/rc 这个档案就知道为什幺了
所以当你要使用第三或第四个档案时请自己建立, 就像妳写个 shell 一样很简单
```bash
cat /etc/init.d/after.local
#!/bin/bash  
/etc/init.d/sendip start  
```



