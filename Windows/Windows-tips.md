# Windows 下技巧

[TOC]

## 删除保存的samba密码
```batch
net  use  *  /del
```

## windows下上帝模式
建立一个文件夹 命名如下
```batch
GodMode.{ED7BA470-8E54-465E-825C-99712043E01C} 
```

## 修改hosts
C:\Windows\System32\drivers\etc\hosts
```
    10.239.128.63       S63
    10.239.128.61       S61
    10.239.129.59       S59
    10.239.128.181      S181
    10.239.173.22       SC
    10.239.173.17       SU
```

## 添加右键sublime打开
```
regedit --> HKEY_CLASSES_ROOT\*\shell
new Key: Open with sublime
--> Open with sublime
new Key: command

C:\Users\xxx\sublime_text.exe "%1"
```

还可以添加icon
在上面的 "Open with sublime" 右键 "new" --> "String Value"
value name: icon
value data: ico 的路径

## 创建快捷方式
```bat
mklink /j VSCode VSCode-win32-x64-1.34.0
```

`/d`: 目录的快捷方式 但是权限不够
`/j`: Junction的快捷方式

## 开机启动脚本
C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
> 上面的 **用户名** 根据自己的实际情况填写

或者：Win + R --> `shell:startup` 打开 Startup 目录

把要开机启动的脚本放到 Startup[^Startup] 目录下

[^Startup]: https://blog.51cto.com/u_15338624/3596049#:~:text=1.%E5%BC%80%E5%A7%8B%2D%3E%E8%BF%90%E8%A1%8C%2D,%E8%BF%99%E6%A0%B7%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BA%86%E3%80%82

