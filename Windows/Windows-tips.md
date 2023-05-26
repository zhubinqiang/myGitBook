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
注册表[^registry]包含在操作期间 Windows 持续引用的信息，
例如每个用户的配置文件、计算机上安装的应用程序
以及每个用户可以创建的文档类型、文件夹
和应用程序图标的属性表设置、系统上存在的硬件以及正在使用的端口。


|   文件夹/预定义项   |                  说明[^regedit]                  |
| :------------------ | :----------------------------------------------- |
| HKEY_CLASSES_ROOT   | 存储可打开文件的类型、扩展名等，适用于所有用户   |
| HKEY-CURRENT-USER   | 保存登录用户设置、控制面板选项、映射的网络驱动器 |
| HKEY-LOCAL-MACHINE  | 包含机器的所有硬件和文件上安装的软件信息         |
| HKEY-USER           | 保存所有登录在此机上用户的信息，如自定义桌面等   |
| HKEY-CURRENT-CONFIG | 连接到计算机上的硬件配置数据，如显示器、打印机等 |


```ini
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Open with nvim]
"icon"="C:\\Users\\YOURNAME\\WS\\icons\\Nvim.ico"

[HKEY_CLASSES_ROOT\*\shell\Open with nvim\command]
@="C:\\Users\\YOURNAME\\Program\\nvim-win64\\bin\\nvim-qt.exe \"%1\""

[HKEY_CLASSES_ROOT\*\shell\Opwn with VSCode]
"icon"="C:\\Users\\YOURNAME\\WS\\icons\\microsoft_visual_studio_code_icon_256x256.ico"

[HKEY_CLASSES_ROOT\*\shell\Opwn with VSCode\command]
@="C:\\Users\\YOURNAME\\Program\\VSCode\\Code.exe \"%1\""



[HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\office\16.0\outlook\pst]
"PSTDisableGrow"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters]
"AllowInsecureGuestAuth"=dword:00000001


[HKEY_CURRENT_USER\Environment]
"JAVA_HOME"="C:\\Users\\YOURNAME\\Program\\microsoft-jdk-17.0.1.12.1-windows-x64"
"MyLinks"="C:\\Users\\YOURNAME\\MyLinks"
"NODE_HOME"="C:\\Program Files\\nodejs"
"NVM_HOME"="C:\\Users\\YOURNAME\\AppData\\Roaming\\nvm"
"PYTHON_HOME"="C:\\Users\\YOURNAME\\AppData\\Local\\Programs\\Python\\Python310"
```

## AppData文件夹
AppData[^appdata] 文件夹下有Local，Locallow和Roaming

`appdata` ==> C:\Users\YOURNAME\AppData
`%appdata%` ==> C:\Users\YOURNAME\AppData\Roaming
`%localappdata%` ==> C:\Users\YOURNAME\AppData\Local

Roaming 文件夹是一种可以轻松与服务器同步的文件夹。
它的数据可以随着用户的个人资料从PC移动到PC — 就像当您在域中时，您可以轻松登录到任何计算机并访问其收藏夹、文档等。

Local文件夹主要包含与安装程序相关的文件夹。
其中包含的数据 (`%localappdata%`) 无法随您的用户配置文件一起移动，因为它特定于PC，因此太大而无法与服务器同步。



## 创建快捷方式
```bat
mklink /j VSCode VSCode-win32-x64-1.34.0
```

`/d`: 目录的快捷方式 但是权限不够
`/j`: Junction的快捷方式

## 开机启动脚本
C:\Users\YOURNAME\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
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
[^registry]: https://learn.microsoft.com/zh-cn/troubleshoot/windows-server/performance/windows-registry-advanced-users
[^regedit]: https://blog.csdn.net/beiback/article/details/125589844
[^appdata]: https://zhuanlan.zhihu.com/p/557663690
