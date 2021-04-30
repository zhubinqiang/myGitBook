# Windows batch 脚本

[TOC]

对 [简书](https://www.jianshu.com/p/dbb7b13c5ed8) 作者：ettingshausen 做的摘录

## 查看脚本
```bat
type myScript.bat

more myScript.bat
```

## 运行脚本
```bat
%COMSPEC% /D /C  "C:\Users\User\SomeScriptPath.cmd" Arg1 Arg2 Arg3
```
`/C`: 在脚本运行完成后退出子进程
`/D`: 禁止所有自动运行的脚本。这样做的原因是可以阻止命令提示窗口自动关闭，如果我写的脚本或者调用的脚本中调用了 `EXIT` 命令。`EXIT` 命令会自动关闭命令提示符窗口，除非是从子命令提示符进程中调用 `EXIT`。

## 注释
1. REM (Remark)
```bat
REM this is comment
```

2. 使用 `::`，两个冒号
```bat
:: This is also a comment too! (usually!!)
```

批处理文件中的第一个非注释行，通常是一条关闭输出的命令
```bat
@echo off
```

`@` 是一个特殊的操作符，用于抑制命令行的打印。一旦我们关闭了输出，在后续的脚本命令中就不再需要 `@` 操作符了。可以使用以下命令恢复打印:
```bat
echo on
```

## 变量
### 使用set赋值
```bat
set foo=bar
```

> **Note:** =左右不能有空格

`/A` 开关支持算数操作
```bat
set /A four=2+2
echo %four%
```

`/P` 输出提示信息, 终端输入后赋值给变量
```bat
set /P txt="input: "
echo "%txt%"
```

一般的惯例，变量经常使用小写名称，系统变量（环境变量）一般是大写。DOS是不区分大小写的，所以这个惯例并不是强制的。

> **Note:**
> 1. DOS包含一些“动态”的环境变量，它们的行为更像是命令。这些动态变量包括 `%temp%`、 `%DATE%`、`%RANDOM%`和 `%CD%`。最好不要覆盖这些动态变量。
> 2. `ECHO %foo%` 可以快速确认 foo 是不是一个现有的变量。

`SET` 命令不加参数，会输出所有的变量到控制台。这些变量大多数是环境变量，比如 `%PATH%` 或者 `%TEMP%`。

### 动态环境变量
调用 `SET` 将列出当前会话的所有常规（静态）变量。**不包括动态环境变量**，如 `%DATE%` 或 `%CD%`。在SET帮助文本的末尾列出这些动态变量，可以通过调用 `SET /?` 来查看。

|         动态变量           |              含义               |
| :------------------------ | :------------------------------ |
| `%CD%`                    | 当前目录字符串                  |
| `%DATE%`                  | 当前日期                        |
| `%TIME%`                  | 当前时间                        |
| `%RANDOM%`                | 0 到 32767 之间的任意十进制数字 |
| `%ERRORLEVEL%`            | 当前 ERRORLEVEL 数值            |
| `%CMDEXTVERSION%`         | 当前命令处理器扩展版本号        |
| `%CMDCMDLINE%`            | 调用命令处理器的原始命令行      |
| `%HIGHESTNUMANODENUMBER%` | 此计算机上的最高 NUMA 节点号    |

### 变量作用范围
**默认情况下，变量对整个命令提示符会话是全局的**。

global.bat
```bat
set v=global
echo %v%
```

调用`SETLOCAL`命令，将变量变为局部变量。任何局部变量赋值在调用`ENDLOCAL`，`EXIT`，或者当执行到达脚本中的文件结尾（EOF）时都会恢复。
local.bat
```bat
setlocal
set v=local
echo %v%
endlocal
```

当前有2个bat脚本，在cmd当中
```bat
global.bat

:: global
echo %v%

local.bat

:: global
echo %v%
```

### 命令行参数技巧
#### `%~I` 从第`I`个命令行参数中删除引号
`%~I` 从第`I`个命令行参数中删除引号，在处理文件路径参数时非常有用。带空格的路径需要用引号括起来，但是多次括起来就会导致错误。这里的I可以事`0~9`的整数。

#### `%~fI` 完整路径
a.bat
```bat
echo %~f1
```

```bat
a.bat xxx.txt

:: C:\Users\user\xxx.txt
```

#### `%~fsI` 
`%~fsI`与上边的类似，s 选项会生成一个 DOS 8.3[1]的短路径，例如：`C:\Program Files`缩写为 `C:\PROGRA~1`。在使用一些不处理空格的第三脚本。

#### `%~dpI` 第`I`个文件路径参数的完整父级路径
`%~dpI` 第I个文件路径参数的完整父级路径。`SET parent=%~dp0`通过这个用法，可以输出脚本的所在路径。

#### `%~nxI` 第`I`个文件路径参数的文件名（包括扩展名）。
类似`%0`， 运行时脚本时 `ECHO %~n0 message`: 输出的消息，而不是直接输出ECHO 输出的消息。这样做的好处是，用户知道这个消息是从哪个脚本中输出的。


#### 批处理的参数
```bat
%0 批处理文件本身
%1 第一个参数
%9 第九个参数
%* 从第一个参数开始的所有参数
```


有用的
```bat
SETLOCAL ENABLEEXTENSIONS
SET me=%~n0
SET parent=%~dp0
```

`SETLOCAL`命令能保证脚本在退出之后不覆写任何现有的变量。`ENABLEEXTENSIONS`，是SETLOCAL的一个参数，这是一个非常有用的特性，称之为启动或停用命令处理器扩展名。
`me`变量存储了脚本的名称（不包含扩展名）。这样就能很方便的给输出的消息加上前缀`ECHO %me%: 输出的消息`。
`parent` 存储了脚本的所在目录，这样可以很容易的给同一目录下的其他文件拼接出完整的路径。

## 字符
下面时常规的字符串操作[^string]

### 截取字符串
```bat
set testStr=abcdefghijklmnopqrstuvwxyz0123456789
echo 原始字符串 %testStr%
echo 提取前五个字符串：%testStr:~0,5%
echo 提取最后五个字符串：%testStr:~-5%
echo 提取第一个到倒数第六个字符串：%testStr:~0,-5%
```

### 字符串替换
将字符串里面的needs替换成has
```bat
set str=This message needs changed. 
echo %str% 

set str=%str:needs=has% 
echo %str%
```

### 字符串合并
```bat
set aa=aabbcc
set bb=ddeeff
set "aa=%aa%%bb%"
echo aa=%aa%
```

### 一行太长换行
```bat
echo ^
hello ^
world
pause
```

### 字符串扩充
“扩充”这个词汇来自于微软自己的翻译，意思就是对表示文件路径的字符串进行特殊的处理。
```bat
%~1          - 删除引号(")，扩充 %1
%~f1         - 将 %1 扩充到一个完全合格的路径名
%~d1         - 仅将 %1 扩充到一个驱动器号
%~p1         - 仅将 %1 扩充到一个路径
%~n1         - 仅将 %1 扩充到一个文件名
%~x1         - 仅将 %1 扩充到一个文件扩展名
%~s1         - 扩充的路径指含有短名
%~a1         - 将 %1 扩充到文件属性
%~t1         - 将 %1 扩充到文件的日期/时间
%~z1         - 将 %1 扩充到文件的大小
%~$PATH : 1  - 查找列在 PATH 环境变量的目录，并将 %1
               扩充到找到的第一个完全合格的名称。如果环境
               变量名未被定义，或者没有找到文件，此组合键会
               扩充到空字符串

%~dp1        - 只将 %1 扩展到驱动器号和路径
%~nx1        - 只将 %1 扩展到文件名和扩展名
%~dp$PATH:1 - 在列在 PATH 环境变量中的目录里查找 %1，
               并扩展到找到的第一个文件的驱动器号和路径。
%~ftza1      - 将 %1 扩展到类似 DIR 的输出行。
```

## 标准输入输出
```bat
DIR > temp.txt
DIR >> temp.txt
```

标准错误重定向种 2 与 `>>` 没有空格
```bat
DIR SomeFile.txt 2>> error.txt
Some.exe 2> error.txt 1>&2
```

使用 `<` 从文件读到脚本
```bat
type data.txt
2
3
1
5
4

sort < data.txt > result.txt
```

丢弃可以定向到 `NUL`
```bat
ping 127.0.0.1 > NUL
```

清空文件内容
```bat
type NUL > output.txt
```

管道符 `|`
```bat
dir /b | sort
```

终端种创建文本，当输入 `Ctrl + C` 退出
```bat
type con > output.txt
```

## if 语句
### if 一行语句
```bat
if exist "temp.txt" echo found

if not exist "temp.txt" echo not found
```

### if ... else ...
```bat
if exist "data.txt" (
    echo "found"
) else (
    echo "not found"
)
```

> **Note:** `if` 和 `else` 条件后的执行语句，只能做一条语句看待。 多行时使用 `()`
> 批处理读取命令时是按行读取的（另外例如for命令等，其后用一对圆括号闭合的所有语句也当作一行）

判断字符串是否为空
```bat
if "%var%"=="" (
    set var=12345
)

if not defined var (
    set var=12345
)
```

### 判断字符串是否相等
```bat
set var=Hello, World
if "%var%"=="Hello, world" (
    echo found
) else (
    echo not found
)
```

### `if /i` 忽略字符串大小写
```bat
if /i "%var%"=="hello, world" (
    echo found
) else (
    echo not found
)
```

### 数值判断
```bat
set /a var=0
if /i "%var%" equ "1" (
    echo "1"
) else (
    if /i "%var%" lss "1" (
        echo "less than 1"
    ) else (
        echo "greater than 1"
    )
)
```

```bat
set /a var=0
if /i "%var%" equ "1" (
    echo "1"
) else if "%var%" lss "1" (
    echo "less than 1"
) else (
    echo "greater than 1"
)
```

| 操作符 |         意思          | 数学符号 |
| :----- | :-------------------- | :------- |
| EQU    | equal                 | =        |
| NEQ    | not equal             | !=       |
| LSS    | less than             | <        |
| LEQ    | less than or equal    | <=       |
| GTR    | greater than          | >        |
| GEQ    | greater than or equal | <=       |

### 判断返回值
```bat
if /i %ERRORLEVEL% neq "0" (
    echo "execution failed"
)
```

## 循环语句
for 的在bat种的基本语句：
```bat
FOR %%variable IN (set) DO command
```

`FOR` 命令使用一个特殊的变量语法`%`，后跟一个字母，如`%I`。当批处理文件中使用此语法时，略有不同，需要两个百分号`%%I`。在编写脚本时，这是一个常见的错误来源。如果for循环因为语法错误退出，确认是否使用了`%%I`。

**参数是大小写敏感的**，`%%I` 和 `%%i` 是不一样的

这种在命令行里面是可以运行的，但是在bat脚本里面是错误的。
```bat
for %i in (%USERPROFILE%\*) do @echo %i
```

这种在bat脚本里面是正确的
```bat
for %%i in (%USERPROFILE%\*) do @echo %%i
```

### `/L` 控制循环次数
```bat
FOR /L %%variable IN (start,step,end) DO command
```

奇数行输出
```bat
for /L %%i in (1,2,10) do (
    echo %%i
)
```

倒叙输出
```bat
for /L %%i in (5,-1,1) do (
    echo %%i
)
```

### `/D` 遍历文件夹
```bat
for /D %%i in (%USERPROFILE%\*) do (
    echo "%%i"
)
```

### `/R` 遍历文件
```bat
for /R "%CD%" %%i in (*) do (
    echo "%%i"
)

:: all folders
for /R "%CD%" /D %%i in (*) do (
    echo "%%i"
)
```

### `/F` 对字符串，命令的返回值，访问硬盘上的ASCII码文件
格式为：
```bat
FOR /F ["options"] %%variable IN (set) DO command
```

其中，set为("string"、'command'、file-set)中的一个；
options是(eol=c、skip=n、delims=xxx、tokens=x,y,m-n、usebackq)中的一个或多个的组合。
一般情况下，使用较多的是skip、tokens、delims三个选项。

读取文件内容
```bat
for /F %%i in (data.txt) do (
    echo %%i
)
```

`/F` 会默认以每一行来作为一个元素

如果我们还想把每一行再分解更小的内容, 使用 `delims` 和 `tokens`

```bat
for /F "tokens=2,3 delims= " %%i in (result.txt) do (
    echo %%i %%j
)
```
`delims=` 以空格为为分隔符. delims的值默认为空格
`tokens=2,3` 读取2，3两列。`tokens=2-4` 2到4列

怎么多出一个 `%%j` ？
这是因为你的tokens后面要取每一行的两列，用 `%%i` 来替换第二列，用 `%%j` 来替换第三列。
**并且必须是按照英文字母顺序排列的，%%j不能换成%%k，因为i后面是j**

`skip=2` 跳过前2行
```bat
for /F "skip=2 tokens=2-4 delims= " %%i in (result.txt) do (
    echo %%i %%k
)
```

`eol=2` 跳过以 `2` 开头的行
```bat
for /F "eol=2 tokens=2-4 delims= " %%i in (result.txt) do (
    echo %%i %%k
)
```

set 为 `"string"` 时
```bat
for /F "tokens=2-4 delims=," %%a in ("1,2,3,4,5") do (
    echo %%a %%c
)
```

set 为 `'command'` 时
```bat
for /F "tokens=3* delims= " %%a in ('dir') do (
    if not "%%a"=="<DIR>" (
        echo %%a %%b
    )
)
```

列表 [^list]
```bat
set list=1 2 3 4 
for %%a in (%list%) do ( 
   echo %%a 
)
```

数组
```bat
@echo off
setlocal enabledelayedexpansion
set topic[0]=comments
set topic[1]=variables
set topic[2]=Arrays
set topic[3]=Decision making
set topic[4]=Time and date
set topic[5]=Operators

for /l %%n in (0,1,5) do (
   echo !topic[%%n]!
)
```

## 函数
有两个需要注意的问题：

1. “函数”需要在**脚本的底部**通过标签来定义
2. 脚本的主要逻辑（主函数）需要 `EXIT /B [errorcode]` 这样可以组织主逻辑进入函数。
3. 结束时`EXIT /B`关键字。遗憾的是，除了退出代码之外，什么都不能返回

```bat
@ECHO OFF
SETLOCAL

set me=%~n0
set log=%TEMP%\%me%.txt

CALL :show %log%
CALL :tee "%me%: Hello, world!"
CALL :show %log%

EXIT /B %ERRORLEVEL%


:tee
echo %* >> "%log%"
echo %*
EXIT /B 0

:show
type %*
EXIT /B 0
```

## 进阶
### 延迟环境变量扩展
```bat
@echo off 
set a=4 
set a=5 & echo %a% 
```
上面输出是4

**批处理读取命令时是按行读取的（另外例如for命令等，其后用一对圆括号闭合的所有语句也当作一行）**，
在处理之前要完成必要的预处理工作，这其中就包括对该行命令中的变量赋值。

批处理在运行到这句`set a=5 & echo %a%`之前，先把这一句整句读取并做了预处理 对变量`%a%`从上面的语句寻找 `set a=`的值并替换掉，那么`%a%`是4了！预处理后的结果是 `set a=5 & echo 4`. 然后就是把这条语句执行一次。


```bat
@ECHO OFF
SETLOCAL
for /l %%i in (1, 1, 3) do (
    set var=%%i
    echo %var%
)
```
上面得到3条 ECHO is off.

批处理在运行到 `for /l %%i in (1, 1, 3) do (set var=%%i & echo %var%)` 之前, 
先把这一句整句读取并做了预处理 对变量var从此处的for之前的语句 找寻 `set var=` 并替换掉。
for之前并有`set var=` 所有为空。


```bat
@echo off
setlocal enabledelayedexpansion
set a=4
set a=5 & echo !a!
```

“setlocal enabledelayedexpansion[^enabledelayedexpansion]”开启变量延迟，然后`set a=4`先给变量a赋值为4，
`set a=5 & echo !a!` 这句是给变量a赋值为5并输出（由于启动了变量延迟，所以批处理能够感知到动态变化，即不是先给该行变量赋值，而是在运行过程中给变量赋值，因此此时a的值就是5了）

**在延迟变量扩展中，要使用 `!` 来引用变量**

这个可以得到正确输出
```bat
@ECHO OFF
SETLOCAL ENABLEDELAYEDEXPANSION
for /l %%i in (1, 1, 3) do (
    set var=%%i
    echo var=!var!, i=%%i
)
```

[^string]: 作者:pengcao89 链接: https://blog.csdn.net/peng_cao/article/details/74170979
[^list]: 教程: https://www.tutorialspoint.com/batch_script/batch_script_arrays.htm
[^enabledelayedexpansion]: 作者:mdxy-dxy 链接: https://www.jb51.net/article/29323.htm

