# Perl 笔记
[TOC]

## 注释
```perl
=pod
1.xxx
2.ddd
3.这是多行注释
=cut

# this is comment 单行注释
```

## 判断语句

### if ... elsif ... else ...
```perl
my @array = (1..10);
foreach (@array) {
    if($_ == 1) {
        print "This is one($_)\n";
    } elsif($_ == 2) {
        print "This is two($_)\n";
    } elsif($_ == 3) {
        next;              # 跳到下一次 相当于其他语言中的continue
    } else {
        print "$_\n";
    }

    if($_ > 5) {
        last;               # 跳出循环
    }
}
```

### unless ... else ...
```perl
my $i = 1;
unless($i > 5){
    say "$i is not larger than 5";
}else {
    say "$i is larger than 5";
}
```
if和unless的区别：***if(condition)  ==  unless(! condition)***

### Perl 判断值为true或false的规则

 1. Perl 不是用的这些规则，但可以利用它们方便记忆，其结果是一致的。
 2. 如果值为数字，0 是 false；其余为真。
 3. 如果值为字符串，则空串('')为 false；其余为真。
 4. 如果值的类型既不是数字又不是字符串，则将其转换为数字或字符串后再利用上述规则。
 5. undef为 false。所有的引用都是 true。
 
**这些规则中有一个特殊的地方。 由于字符串 '0' 和数字 0 有相同的标量值，Perl 将它们相同看待。也就是说字符串 '0' 是唯一一个非空但值为0的串。**


## 循环语句

### while
```perl
$i = 1;
while($i < 10){
    print $i."\n";
    $i += 1;
}
```

### while ... continue...
```perl
#!/usr/bin/env perl

my $i = 1;
while($i < 5) {
    print "$i \n";
}continue {
    $i++;
}
```

### until
```perl
my $i = 1;
until($i >=5 ) {
    print "$i \n";
    $i++;
}
```
while 和 until的区别： ***while(condition)  ==  until(! condition)***

### do ... while ...
```perl
my @a = (1 .. 10);
my $i;
do {
    $i = shift @a;
    say $i;
} while ($i < 5);
```
***Perl 中没有 do 。。。while。。。循环语句， 实际上这是do语句与while语句一起工作。***

### for
```perl
for($i=1; $i<=9; $i++){
    for($j=1; $j<=$i; $j++){
        print "$j*$i=".($i*$j)."\t";
    }
    print "\n";
}
```


### foreach
```perl
@list = (1 .. 5);
foreach my $i(@list){
    print $i."\n";
}

foreach (@list){
    print "$_\n";
}
```

### 控制循环的三个关键字: next, last, and redo
next 像C，java语言中的continue语句。跳过余下的代码块，进入循环中的下一个值。
```perl
while(<FILE>) {
    next if /^#/;   # 假如是空行，就读下一行。
    print "$_";
}
```

last像其他语言中的break语句。直接退出整个循环。
```perl
while(<FILE>) {
    last if /^End#/;  # 匹配End就 退出循环
    print "$_";
}
```

redo再一次执行本次循环
```perl
foreach(1 .. 5) {
    print "$_\n";
    chomp(my $ch=<STDIN>);
    if($ch =~ /a/) {
        redo;      # 当输入a是，再执行本次循环
    } 
}
```

举个例子
```perl
 foreach(1 .. 5) {
     print "Number:$_\n";
     print "input: last,next, redo, or none of them\n";
     chomp(my $ch=<STDIN>);
     last if($ch =~ /last/i);
     next if($ch =~ /next/i);
     redo if($ch =~ /redo/i);
     print "That wasn't any of the choices\n";
 }
 print "That's all\n";

```


----------

运行结果：
Number:1
input: last,next, redo, or none of them
next
Number:2
input: last,next, redo, or none of them
abc
That wasn't any of the choices
Number:3
input: last,next, redo, or none of them
redo
Number:3
input: last,next, redo, or none of them
redo
Number:3
input: last,next, redo, or none of them
last
That's all

----------


###使用标签
在内部和外部的循环使用标签退出内部循环。
```perl
OUTER:
   while(<DATA>) {
      chomp;
      @linearray = split;
      foreach $word (@linearray) {
         next OUTER if($word =~ /next/i);
      }
   }
```

**在循环过程中 `$_`  为默认参数**
```perl
foreach(1 .. 10) {
    print "$_\n";
}
```

## Array
```perl
my @array = (1, 2, 3, 4, 5);
print $array[0];               # 打印第0个元素
print $array[1];               # 打印第1个元素
print $array[$#array];         # 打印最后一个元素

foreach my $i (@array) {       # 遍历整个Array
    print "$i\n";
}
```

### 数组赋值
```perl
$array[0] = “a”;               # 为地0个元素赋值
```

### 数组操作
#### push
往数组最末尾添加元素
```perl
my @temp;
push(@temp, 'a');
push(@temp, 'b');
push(@temp, 5);
push(@temp, 8);
print @temp;     # 输出 ab58

```

#### pop
从数组的末尾取得元素，并删除数组中该元素
```perl
my @temp = (1, 2, 3, 4, 5);
my $p1 = pop(@temp);   # $p1 = 5;
my $p2 = pop(@temp);   # $p2 = 4;
```

#### shift
从数组的最前头取得元素，并删除数组中该元素
```perl
my @temp = (1, 2, 3, 4, 5);
my $p1 = shift(@temp);   # $p1 = 1;
my $p2 = shift(@temp);   # $p2 = 2;
```

#### unshift
往数组最前头添加元素
```perl
my @temp = (1, 2, 3, 4, 5);
unshift(@temp, 'a');
unshift(@temp, 'b');
print @temp;         # 输出 ba12345;
```

#### 分割数组
```perl
my @s1 = qw(a b c d e f);
my @s2 = splice(@s1, 2);
print @s1;      # ab
print @s2;      # cdef
```

#### qw 函数
qw(foo bar baz) 相当于 ('foo', 'bar', 'baz')
可以使用任何分隔符，而不仅仅是括号组。
```perl
my @a = qw(hello world I am here);
my @b = qw {
    /sbin
    /usr/sbin
    /usr/local/sbin
};

foreach(@a) {
    print "$_ \n";
}
```

### 数组中 相同元素和不同元素
```perl
my @array = ( 'a', 'b', 'c', 'a', 'd', 1, 2, 5, 1, 5 );
my %diff;
my %same;
my @diff = grep(!$diff{$_}++, @array);
my @same = grep($same{$_}++, @array);
```

## Hash
第一种和第二种是一样的， 但第二种更直观。
第一种以(k1, v1, k2, v2, k3, v3)方式存在 
```perl
my %h1 = ("dog", "1", "cat", "2", "fish", "3");
my %h2 = (
    "dog" => '1',
    "cat" => '2',
    "fish" => '3'
);
print $h1{'dog'};
print $h1{dog};
```

### Hash赋值, 删除, 拷贝
```perl
$h1{'cow'} = '4';   #赋值
$h2{cow} = '4';     #赋值
delete( $h1{'dog'} );   #删除
%h3 = %h1;   #拷贝
```

### Hash遍历
#### keys函数遍历Hash
keys 函数得到hash中所有key的数组
values 函数得到hash中所有的values的数组
```perl
my %h1 = (
    "dog" => '1',
    "cat" => '2',
    "fish" => '3'
);
my @animals = keys %h1;
foreach(@animals){
    print "$_ => $h{$_}\n";
}

my @numbers = values %h1;
foreach my $n(@numbers){
    print "$_\n";
}
```

可以用sort函数对key进行排序
```perl
foreach(sort keys %h1){
    print $_;
}
```

#### each函数遍历Hash
```perl
my %h1 = (
    "dog" => '1',
    "cat" => '2',
    "fish" => '3'
);
while(my($k, $v) = each(%h1)){
    print "$k => $v \n";
}
```

### 检测存在
exists函数来检测是否存在
```perl
my %h1 = (
    "dog" => '1',
    "cat" => '2',
    "fish" => '3'
);

if(exists($h1{"dog"})) {
    print "dog exist!";	
}else {
    print "dot doesn't exist!";
}
```

哈希结构的2级哈希使用匿名散列引用
```perl
my $variables = {

   scalar  =>  { 
      description => "single item",
      sigil => '$',

     },
   array   =>  {
      description => "ordered list of items - www.yiibai.com",

      sigil => '@',
     },
   hash    =>  {

      description => "key/value pairs",
      sigil => '%',
     },

};

print "$variables->{'scalar'}->{'sigil'}\n";
```

## 文件I/O
### 文件测试
| 选项 | 含义 |
|-----|:------|
|-r |文件或目录对此（有效的）用户（effective user）或组是可读的
|-w |文件或目录对此（有效的）用户或组是可写的
|-x |文件或目录对此（有效的）用户或组是可执行的
|-o |文件或目录由本（有效的）用户所有
|-R |文件或目录对此用户(real user) 或组是可读的
|-W |文件或目录对此用户或组是可写的
|-X |文件或目录对此用户或组是可执行的
|-O |文件或目录由本用户所有
|-e |文件或目录名存在
|-z |文件存在，大小为 0（目录恒为 false）
|-s |文件或目录存在，大小大于 0（值为文件的大小，单位：字节）
|-f |为普通文本
|-d |为目录
|-l |为符号链接
|-S |为 socket
|-p |为管道(Entry is a named pipe(a “ fifo” ))
|-b |为 block -special 文件（如挂载磁盘）
|-c |为 character -special 文件（如 I/O 设备）
|-u |setuid 的文件或目录
|-g |setgid 的文件或目录
|-k |File or directory has the sticky bit set
|-t |文件句柄为 TTY( 系统函数 isatty() 的返回结果；不能对文件名使用这个测试)
|-T |文件有些像“文本”文件
|-B |文件有些像“二进制”文件
|-M |修改的时间（单位：天）
|-A |访问的时间（单位：天）
|-C |索引节点修改时间（单位：天）


举个列子
```perl
my $f = "/home/user/a.txt";
if(-e $f) {
    print "$f does exist!";
    if(-f $f) {
        print "$f is a file";
    }elsif(-d $d) {
        print "$f is a directory";
    }
}else {
    print "$f not found !";
}

```

```perl
use 5.010;
sub readFile{
    my ($filepath) = @_;

    say $filepath;
    open FH, "<$filepath";
    my @mytext = <FH>;
    foreach my $text(@mytext){
        #say $text;
        my @line = split(":", $text);
        say $line[0];
    }

    close FH;
}

$filepath = "passwd";
&readFile($filepath);

## 第二种  
open my $f, "<" , "f1.pl" or die "can not open $!";
while(<$f>){  
    print $_;
}

close $f;

## 写文件
sub f1 {
    open(F, ">a.txt") or die "can't open $!";  
    print F "123456789\n";  
    close F;  
}
```












## Perl 内置操作符
### 算术运算符
| 操作符 | 含义 |
|-----|:------|
|+ |加
|- |减
|* |乘
|/ |除


### 数字比较运算符
| 操作符 | 含义 |
|-----|:------|
|==  |等于
|!=  |不等于
|<   |小于
|>   |大于
|<=  |小于等于
|\>= |大于等于

### 字符串比较运算符
| 操作符 | 含义 |
|-----|:------|
|eq  |equality
|ne  |inequality
|lt  |less than
|gt  |greater than
|le  |less than or equal
|ge  |greater than or equal

### 杂项运算符
| 操作符 | 含义 |
|-----|:------|
| .  |字符串连接 "abc" . "123"
| x  |字符串多次连接 "abc" x 5
| .. |范围 (1 .. 10)

## Perl 内部变量
| 变量 | 含义 |
|-----|:------|
|$- |当前页可打印的行数,属于Perl格式系统的一部分
|$! |根据上下文内容返回错误号或者错误串
|$” |列表分隔符
|$# |打印数字时默认的数字输出格式
|$$ |Perl解释器的进程ID
|$% |当前输出通道的当前页号
|$& |与上个格式匹配的字符串
|$( |当前进程的组ID
|$) |当前进程的有效组ID
|$* |设置1表示处理多行格式.现在多以/s和/m修饰符取代之.
|$, |当前输出字段分隔符
|$. |上次阅读的文件的当前输入行号
|$/ |当前输入记录分隔符,默认情况是新行
|$: |字符设置,此后的字符串将被分开,以填充连续的字段.
|$; |在仿真多维数组时使用的分隔符.
|$? |返回上一个外部命令的状态
|$@ |Perl解释器从eval语句返回的错误消息
|$[ |数组中第一个元素的索引号
|$\ |当前输出记录的分隔符
|$] |Perl解释器的子版本号
|$^ |当前通道最上面的页面输出格式名字
|$^A |打印前用于保存格式化数据的变量
|$^D |调试标志的值
|$^E |在非UNIX环境中的操作系统扩展错误信息
|$^F |最大的文件捆述符数值
|$^H |由编译器激活的语法检查状态
|$^I |内置控制编辑器的值
|$^L |发送到输出通道的走纸换页符
|$^M |备用内存池的大小
|$^O |操作系统名
|$^P |指定当前调试值的内部变量
|$^R |正则表达式块的上次求值结果
|$^S |当前解释器状态
|$^T |从新世纪开始算起,脚步本以秒计算的开始运行的时间
|$^W |警告开关的当前值
|$^X  |Perl二进制可执行代码的名字
|$_  |默认的输入/输出和格式匹配空间
|\$\|  |控制对当前选择的输出文件句柄的缓冲
|$~ |当前报告格式的名字
|$`  |在上个格式匹配信息前的字符串
|$’ |在上个格式匹配信息后的字符串
|$+ |与上个正则表达式搜索格式匹配的最后一个括号
|$< |当前执行解释器的用户的真实ID
|$< digits > |含有与上个匹配正则表达式对应括号结果
|$= |当前页面可打印行的数目
|$> |当前进程的有效用户ID


## 包含正在执行的脚本的文件名
| 变量 | 含义 |
|-----|:------|
|$ARGV | 从默认的文件句柄中读取时的当前文件名
|%ENV  | 环境变量列表
|%INC  | 通过do或require包含的文件列表
|%SIG  | 信号列表及其处理方式
|@_  | 传给子程序的参数列表
|@ARGV  | 传给脚本的命令行参数列表
|@INC  | 在导入模块时需要搜索的目录列表


## Perl 升级 在CentOS上
```bash
wget -c --no-check-certificate -O - http://install.perlbrew.pl | bash
source ~/perl5/perlbrew/etc/bashrc

#put  "source ~/perl5/perlbrew/etc/bashrc" into ~/.bashrc
grep source '~/perl5/perlbrew/etc/bashrc' ~/.bashrc
if [ "$?" -ne "0" ]; then
    sed -i '$asource ~/perl5/perlbrew/etc/bashrc' bashrc
fi

perlbrew --notest install -Duseshrplib -Dusethreads perl-5.22.0
perlbrew switch perl-5.22.0
wget -c http://xrl.us/cpanm -O ~/perl5/perlbrew/bin/cpanm
chmod a+x ~/perl5/perlbrew/bin/cpanm
~/perl5/perlbrew/bin/cpanm -v Template YAML YAML::XS Date::Calc Archive::Zip
```

