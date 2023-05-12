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

非 administrator `Computer\HKEY_CURRENT_USER\SOFTWARE\Classes\*\shell`
shell这一层可能没有需要自己新建

还可以添加icon
在上面的 "Open with sublime" 右键 "new" --> "String Value"
value name: icon
value data: ico 的路径

## 注册表
```ini
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Open with nvim]
"icon"="C:\\Users\\user\\WS\\icons\\Nvim.ico"

[HKEY_CLASSES_ROOT\*\shell\Open with nvim\command]
@="C:\\Users\\user\\Program\\nvim-win64\\bin\\nvim-qt.exe \"%1\""

[HKEY_CLASSES_ROOT\*\shell\Opwn with VSCode]
"icon"="C:\\Users\\user\\WS\\icons\\microsoft_visual_studio_code_icon_256x256.ico"

[HKEY_CLASSES_ROOT\*\shell\Opwn with VSCode\command]
@="C:\\Users\\user\\Program\\VSCode\\Code.exe \"%1\""



[HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\office\16.0\outlook\pst]
"PSTDisableGrow"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters]
"AllowInsecureGuestAuth"=dword:00000001
```


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

## 解决 unpin from taskbar
打开 `%LOCALAPPDATA%\Microsoft\Windows\Shell\LayoutModification.xml` 这里的xml文件[^unping_from_taskbar]

```xml
  <CustomTaskbarLayoutCollection PinListPlacement="Replace">
    <defaultlayout:TaskbarLayout>
      <taskbar:TaskbarPinList>
		  <!-- <taskbar:DesktopApp DesktopApplicationLinkPath="%APPDATA%\Microsoft\Windows\Start Menu\Programs\System Tools\File Explorer.lnk" /> -->
      </taskbar:TaskbarPinList>
    </defaultlayout:TaskbarLayout>
  </CustomTaskbarLayoutCollection>
```

## 没有 administrator 设置环境变量

非管理员账号设置[^non_admin_env] 环境变量
`Win + R` --> `rundll32.exe sysdm.cpl,EditEnvironmentVariables`

下面这个好像没啥用。
```bat
set _COMPAT_LAYER=RunAsInvoker
start
```

## 解决 “you don’t have appropriate permission to perform this operation”
Refer to https://www.stellarinfo.com/blog/dont-have-appropriate-permission-to-perform-this-operation-in-outlook/
to fix “you don’t have appropriate permission to perform this operation” in Outlook

HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\office\16.0\outlook\pst

`PSTDisableGrow:  1 --> 0`


[^Startup]: https://blog.51cto.com/u_15338624/3596049#:~:text=1.%E5%BC%80%E5%A7%8B%2D%3E%E8%BF%90%E8%A1%8C%2D,%E8%BF%99%E6%A0%B7%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BA%86%E3%80%82
[^non_admin_env]: https://blog.csdn.net/weixin_42005898/article/details/115531523
[^unping_from_taskbar]: https://superuser.com/questions/1251656/items-unpinned-from-taskbar-are-back-after-restart-sign-out-on-windows-10
