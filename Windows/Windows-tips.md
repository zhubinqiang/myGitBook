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
mklink -j VSCode VSCode-win32-x64-1.34.0
```

`/d`: 目录的快捷方式 但是权限不够
`/j`: Junction的快捷方式

