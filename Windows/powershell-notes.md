# Powershell

[TOC]

## 注释
```powershell
# this is comment

<#
mulit line
mulit line
mulit line
#>
```

## 变量
powershell 的变量以 `$` 开头, 不区分大小写
```powershll
$a = 123
$b = 456
$result = $a + $b

$result

## 把 cmdlet命令 赋值给变量
$p = Get-Item env:path

## 查看已使用的变量
ls variable:
ls variable:p*p

## 交换变量
$a,$b = $b,$a

## 给多个变量赋同一个值
$x = $y = $z = 123

## 测试变量是否存在
Test-Path variable:a
Test-Path variable:aa

## 删除变量
del variable:a
```

### 系统变量
```powershell
Get-ChildItem env:

Get-Item env:

dir env:

$env:Path

## powershell 版本
$PSVersionTable
```

## 配置文件[^configuration]
配置文件的路径
```powershell
$PROFILE.AllUsersAllHosts
C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1

$PROFILE.AllUsersCurrentHost
C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.PowerShell_profile.ps1

$PROFILE.CurrentUserAllHosts
~\Documents\WindowsPowerShell\profile.ps1

$PROFILE.CurrentUserCurrentHost
~\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

配置当前用户
```powershell
## 显示 Windows PowerShell 配置文件的路径
## 一般在 ~\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
$profile

test-path $profile

## 创建 Windows PowerShell 配置文件
new-item -path $profile -itemtype file -force
```

配置文件中修改需要再开一个终端才生效. 或者执行 `. ${profile}`时起生效。
```powershell
Set-Alias vi "C:\WINDOWS\vim.bat"
## Remove-Item -Path Alias:vi
##

Set-Alias -Name list -Value Get-Location

function pro { vim $profile }

Function CD32 {Set-Location -Path C:\Windows\System32}
```

> 在 PowerShell 6 删除 alias 使用 `Remove-Alias vi`
> 但在早期版本使用 `Remove-Item -Path Alias:vi`[^remove-alias]


## PowerShell 执行.ps1文件
```
Build.ps1 cannot be loaded because running scripts is disabled on this system.
For more information, see about_Execution_Policies at
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

& ${MSBUILD} -maxCpuCount -target:Build `
    -property:WarningLevel=2 `
    -property:OutDir=bin222\Debug333\ `
    -property:Configuration=Release `
    -property:Platform=x64
```

## if
```powershell
$a = 3

if ($a -gt 2) {
    Write-Host "The value $a is greater than 2."
}
elseif ($a -eq 2) {
    Write-Host "The value $a is equal to 2."
}
else {
    Write-Host ("The value $a is less than 2 or" +
        " was not created or initialized.")
}
```


```powershell
$Folder = 'C:\Windows'
"Test to see if folder [$Folder]  exists"
if (Test-Path -Path $Folder) {
    "Path exists!"
} else {
    "Path doesn't exist."
}
```


## 函数
```powershell
function say_hello {
    $args
    Write-Output("Hello {0}, {1}" -f $args[0], $args[1])
}

## 参数设置默认值
function Say-Hello($name, $count=3) {
    for(; $count -gt 0; $count--) {
        Write-Host("Hello {0}!" -f $name)
    }
}

say_hello abc xyz
Say-Hello abc 2
```

## stop a PowerShell script on the first error
```powershell
Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"
```

## 命令多行
```powershell
$mystring = @"
Bob
went
to town
to buy
a fat
pig.
"@
```


```powershell
$myvar ="Site"
$mystring2 = @"

Bob's $myvar

"@
```

使用反引号
```powershell
& "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin\amd64\MSBuild.exe" `
    -maxCpuCount `
    -t:build `
    -p:Configuration=Release `
    demo.vcxproj
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


[^configuration]: https://forsenergy.com/zh-cn/windowspowershellhelp/html/9c82251c-6f0d-416a-9c3c-77838218531b.htm

[^remove-alias]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/set-alias?view=powershell-7.1#notes


