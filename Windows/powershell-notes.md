# Powershell

[TOC]

## 变量
```powershll
$a = 123

$a
```


## PowerShell 执行.ps1文件
```
Build.ps1 cannot be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at
http://go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ .\Build.ps1
+ ~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

遇到上面的问题是，要 **管理员身份运行** powershell，再执行下面的命令：
```powershell
set-executionpolicy remotesigned
```

上面选择y, 然后再执行 ps1 脚本。


## 运行外部程序
```powershell
${MSBUILD} = "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin\MSBuild.exe"


& ${MSBUILD} -maxCpuCount -target:Build -property:WarningLevel=2 -property:OutDir=bin222\Debug333\ -property:Configuration=Release -property:Platform=x64
```


## Get-Command
```powershell
Get-Command java

Get-Command -ParameterType ServiceController
```


## Get-Service
```powershell
Get-Service -Name w32time | Get-Member

Get-Service -Name w32time | Get-Member -MemberType Method


Get-Service -Name w32time | Select-Object -Property *

Get-Service -Name w32time | Select-Object -Property Status, Name, DisplayName, ServiceType

Get-Service -Name w32time | Select-Object -Property Status, DisplayName, Can*


## (Get-Service -Name w32time).Stop()

```


```powershell
```


