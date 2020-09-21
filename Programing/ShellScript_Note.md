# Shell 笔记
[TOC]

第一个shell script程序
```sh
#!/bin/bash
# Program:
#    this is a test shell script
#    2015/07/10  	zhubinqiang 	Fitst release

echo "Welcome to shell script!"
```

脚本第一行 指定/bin/bash解释器执行，虽然以#开头 但不是注释
> `#`开头的是单行注释

## 注释

```sh
# 这是单行注释1
# 这是单行注释2
```

**Shell里面没有多行注释**，小技巧实现多行注释
```sh
COMMENT_BLOCK=
if [ $COMMENT_BLOCK ]; then
echo "comment2"
echo "comment3"
===========================
fi
```

```sh
<<"COMMENT"
111111111111
222222222222
333333333333
COMMENT
```

## 变量
```sh
a=15
version1=$(uname -r)
version2=`uname -r`
echo $a
echo ${version1}
echo $version2
PATH="$PATH:$HOME/bin"
export PATH
unset a
```

 1. 变量名=变量， 两边没有空格的。 空格在shell中有特殊用途，分开的话会解释成【命令 参数 参数】的形式
 2. 变量名区分大小写。 name 与 Name 是两个变量
 3. 变量有空格的话要用单引号'' 或 双引号 "" 将变量内容结合起来
 4. 上式中\$a 与 \${a} 是相同效果的。使用\${a} 避免出错
 5. '\${a}' 是强引用  输出\${a}。"\${a}"是弱引用，输出变量值
 6. export 使变量成为环境变量
 7. unset 取消变量
 8. shell变量是不区分类型的，变量值都是字符串。Bash允许整数比较和运算操作，但变量中只有数字

## 算术运算
算术运算符

| 操作符    |    描述     |
|:-------- |:------------|
| + | 加法运算 |
| - | 减法运算 |
| * | 乘法运算 |
| / | 除法运算 |
| % | 求余运算 |
| **| 幂运算 |
| +=| 加等于 |
| -=| 减等于 |
| *=| 乘等于 |
| /=| 除等于 |
| %=| 取模等于 |


### let 运算
```sh
a=1
let a=a+1
echo $a           # 2
let a+=1
echo $a           # 3 
let "a = a + 2"
echo $a           # 5
```

### \$((...))运算
```sh
a=1
a=$((a+1))
echo $a
```

### expr 运算
```sh
a=1
b=`expr $a + 1`
echo $b
```

## if ... else ... 判断语句
```sh
a="hello"
if [ "$a" == "hello" ]; then
	echo "Hello, how are you."
elif [ "$a" == "" -o "$a" == '?' ]; then
	echo "what?"
elif [ "$a" == 'y' ] || [ "$a" == 'Y' ]; then
	echo "Yes"
else
	echo "I don't understand you."
fi
```

1. **[ 左右两边有一个空格， == 左右两边有一个空格，] 左边有一个空格**
2. 判断相等可以用一个`=`， **但是 `=` 两边得都有一个空格**, 等号左右两边没有空格是赋值。

### 算术与字符串测试

| 操作    |    描述     |--- --- | 操作   | 描述     |
| :------: |:----------|:------:| :----:  | :--------|
|算术比较  | 中括号[...]结构|   |字符串比较|中括号[...]结构|
|-eq    |     等于   |        |=/==|     等于      |
|-ne    |    不等于  |        | != |    不等于      |
|-lt    |     小于   |        |\<  |   小于(ASCII) *|
|-le    |  小于等于  |        |     |         |
|-gt    |     大于   |        |\\>  |   大于(ASCII)*|
|-ge    |  大于等于  |        |     |         |
|       |           |        | -z   |   字符串为空   |
|       |           |        | -n   |   字符串不为空  |
|算术比较  | 双括号((...))结构|  | 条件判断   |              |
|>      |     大于   |        |-a    | 两个条件同时成立    |
|\>=    |  大小于等于 |        |-o    | 任何一个条件成立    |
|<      |     大于   |        |!     | 非       |
|<=     |  小于等于   |        |      |              |

> 如果在双中括号[[ ... ]]测试中的话，那么不需要转义符\

> **Note**
> 1. 当使用"-n"或者"-z"这种方式判断变量是否为空时，"[ ]"与"[[  ]]"是有区别的。
> 2. 使用"[ ]"时需要在变量的外侧加上双引号，与test命令的用法完全相同，使用"[[  ]]"时则不用。
> 3. 在使用"[[  ]]"时，不能使用"-a"或者"-o"对多个条件进行连接。
> 4. 在使用"[  ]"时，如果使用"-a"或者"-o"对多个条件进行连接，"-a"或者"-o"必须被包含在"[ ]"之内。
> 5. 在使用"[  ]"时，如果使用"&&"或者"||"对多个条件进行连接，"&&"或者"||"必须在"[ ]"之外。
> 6. 在使用符号"=~"去匹配正则表达式时，只能使用"[[  ]]"，当使用">"或者"<"判断字符串的ASCII值大小时，如果结合"[ ]"使用，则必须对">"或者"<"进行转义。

### 文件测试
| 操作    |    描述     |--- --- | 操作   | 描述     |
| :------: |:----------|:------:| :----:  | :--------|
|-e | 文件是否存在 | | -s | 文件大小不为0|
|-f | 是一个标准文件 | |  | |
|-d | 是一个目录 | | -r | 文件具有读的权限|
|-h | 文件是一个符号链接 | | -w | 文件具有写的权限|
|-L | 文件是一个符号链接 | | -x | 文件具有执行权限|
|-b | 文件是一个块设备 | |    | |
|-c | 文件是一个字符设备 | | -g | 设置了sgid标记|
|-p | 文件是一个管道 | | -u | 设置了suid标记|
|-S | 文件是一个socket | | -k | 设置了"粘贴位"|
|-t | 文件与一个终端相关联 | |    | |
|   |  | |    | |
|-N | 从这个文件最后一次被读取之后，它被修改过 | | F1 -nt F2 | 文件F1比文件F2新|
|-O | 这个文件的宿主是你 | | F1 -nt F2 | 文件F1比文件F2旧|
|-G | 文件的组id与你所在的组相同 | | F1 -nt F2 | 文件F1和F2都是同一个文件的硬链接|
|   |  | |    | |
|!  | "非"(翻转上边的测试结果) | |    | |

## Case
像C语言中的switch... case...

```sh
a="hello"
case $a in
"hello")
	echo "Hello, how are you." ;;
"")
	echo "what?" ;;
*)
	echo "I don't understand you." ;;
esac
```

## while 循环
```sh
i=0
s=0
while [ "$i" -le 100 ];do
	s=$((s+i))
	i=$((i+1))
done
echo "1 + 2 + 3 + ... + 100 = $s"
```

## until 循环
```sh
i=0
s=0
until [ "$i" -gt 100 ]; do
	s=$((s+i))
	i=$((i+1))
done
echo "1 + 2 + 3 + ... + 100 = $s"
```

## for 循环
```sh
for i in Tom Jack John; do
	echo $i
done

for i in $(seq 1 5); do
	echo $i
done

for i in {1..5}; do
	echo $i
done

for((i=0; i<5; i++)); do
	echo $i
done
```

## 数组
```sh
array=(5 4 3 2 1)
echo ${array[0]}     # 第一个元素

len=${#array[*]}     # 数组长度
for((i=0; i<$len; i++));do
	echo ${array[$i]}
done

for i in ${!array[@]}; do
	echo ${array[$i]}
done

for i in ${array[@]}; do
	echo $i
done
```

## 函数
**函数定义**
```sh
function function_name {
	command...
}

function function_name() {
	command...
}

function_name() {
	command...
}
```

使用函数
```sh
function sayHello() {
	local name=$1
	echo "Hello $name"
	return 0
}

sayHello "Tom"
```
return返回的是状态码。 0 表示成功， 非0表示失败。

如果要返回值用echo
```sh
function sum(){
	local a=$1
	local b=$2
	local s=$((a+b))
	echo $s
}

s=`sum 2 3`
echo $s
```

用shift 获取参数
```sh
function sum(){
    local a=$1
    shift
    local b=$1
    local s=$((a+b))
    echo $s
}

s=`sum 2 3`
echo $s
```

## I/O操作
终端输入，并赋值
```sh
read -p "Your name(within 30s):" -t 30 named
echo $named
```

输出多行
```sh
cat << EOF
comment1
comment2
comment3
EOF
```

### I/O重定向
```sh
COMMAND > filename             # 重定向stdout到文件filename
COMMAND 1> filename            # 重定向stdout到文件filename
COMMAND >> filename            # 重定向stdout并追加到文件filename
COMMAND 1>> filename           # 重定向stdout并追加到文件filename
COMMAND 2> filename            # 重定向stderr到文件filename
COMMAND 2>> filename           # 重定向stderr并追加到文件filename
COMMAND &> filename            # 将stdout和stderr都重定向到filename
COMMAND &>> filename           # 将stdout和stderr都重定向并追加到filename
COMMAND > filename 2>&1        # 将stdout和stderr都重定向到filename
COMMAND >> filename 2>&1       # 将stdout和stderr都重定向并追加到filename
```

### 管道操作
```sh
COMMAND1 | COMMAND2 | COMMAND3
cat /etc/passwd | grep "root" | awk -F":" '{print $1}' 
```
**管道操作是由前面的stdin传递过来的。 stderr不会传过去**

## 特殊Shell变量
| 变量 | 含义 |
| :--: |:------|
| $0   | 脚本名字 |
| $1   | 位置参数 #1|
| $9   | 位置参数 #9|
| ${10}| 位置参数 #10|
| $#   | 位置参数的个数|
| "$*" | 所有的位置参数(作为单个字符串)|
| "$@" | 位置参数的个数(作为独立的字符串)|
| ${#*}| 传递到脚本中的命令行参数的个数|
| ${#@}| 传递到脚本中的命令行参数的个数|
| $?   | 返回值|
| $$   | 脚本的进程IP(PID)|
| $-   | 传递到脚本中的标志(使用set)|
| $_   | 之前命令的最后一个参数|
| $!   | 运行在后头的最后一个作业的进程ID(PID)|

> "$*" 要用引号引起来，否则默认为 "$@"

## 系统环境变量
| 变量 | 含义 |
| :-------------: |:-------------|
| $IFS | 保存了用于分割输入参数的分割字符，默认识空格|
| $HOME| 当前用户的根目录路径|
| $PATH| 可执行文件的搜索路径|
| $PS1 | 第一个系统提示符|
| $PS2 | 第二个系统提示符|
| $PWD | 当前工作路径|
| $EDITOR| 系统的默认编辑器名称|
| $BASH| 当前Shell的路径字符串|
| $EUID| "有效"用户ID|

## 字符串操作
| 表达式               |    描述     |
| :---------------- |:----------------|
|`${#string}`|`$string`的长度|
|`${string:position}`|在`$string`中，从position开始提取字串|
|`${string:position:length}`|在`$string`中，从position开始取长度为length的字串|
|`${string#substring}`|从变量`$string`的开头，删除最短匹配`$substring`的字串|
|`${string##substring}`|从变量`$string`的开头，删除最长匹配`$substring`的字串|
|`${string%substring}`|从变量`$string`的结尾，删除最短匹配`$substring`的字串|
|`${string%%substring}`|从变量`$string`的结尾，删除最长匹配`$substring`的字串|
|`${string/substring/replacement}`|使用`$replacement`,代替第一个匹配的`$substring`|
|`${string//substring/replacement}`|使用`$replacement`,代替所有匹配的`$substring`|
|`${string/#substring/replacement}`|如果`$string`的前缀匹配`$substring`,那么就用`$replacement`来代替匹配的`$substring`|
|`${string/%substring/replacement}`|如果`$string`的后缀匹配`$substring`,那么就用`$replacement`来代替匹配的`$substring`|
|`expr match "$string" '$substring'`|匹配`$string`开头的`$substring*`的长度 |
|`expr "$string" : '$substring'`|匹配`$string`开头的`$substring*`的长度 |
|`expr index "$string" $substring`|在`$sttring`中匹配到`$substring`的第一个字符出现|
|`expr substr $string $position $length`|在`$string`中从位置`$position`开始提取长度为`$length`的字串|
|`expr match "$string" '\{$substring}\'`|在`$string`的开头位置提取`$substring*`|
|`expr "$string" : '\($substring)\'`|从`$string`的开头位置提取`$substring*`|
|`expr match "$string" '.*\($substring)\'`|从`$string`的结尾提取`$substring*`|
|`expr "$string" : '.*\($substring)\'`|从`$string`的结尾提取`$substring*`|

*** `$substring` 是一个正则表达式**

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

## abcde
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

## 结构汇总
|      操作              |         描述       |
| :-------------------  |:-------------------|
|中括号|`[ ]`|
|`if [ Condition ]` | 测试结构|
|`if [[ Condition ]]` | 扩展的测试结构|
|`Array[1]=element1` | 数组初始化|
|`[a-z]` | 正则表达式的字符范围|
|大括号|`{ }`|
|`${variable}` | 参数替换|
|`${!variable}` | 间接变量引用|
|`{ command1; command2; ... commandN; }` | 代码块|
|`{string1, string2, string3}` | 大括号扩展|
|小括号|`( )`|
|( command1; command2 ) | 子shell中执行的命令组|
|Array=(element1 element2 emement3) | 数组初始化|
|`result=$(COMMAND)` | 在子shell中执行命令，并将结果赋值给变量|
|`>(COMMAND)` | 进程替换|
|`<(COMMAND)` | 进程替换|
|双小括号|`(( ))`|
|`(( var = 78 ))` | 整型运算|
|`var=$(( 20 + 5 ))`| 整型运算，并将结果赋值给变量|
|引号|`' '`|
|"$variable" | "弱"引用|
|'string' | "强"引用|
|后置引用|\` \`|
|result=\`COMMAND\` | 在子shell中运行命令，并将结果赋值给变量|

