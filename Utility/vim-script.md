# vimscript

[TOC]

## 打印
```vim
:echo "Hello again, world!"
:echom "Hello again, world!"

:messages
```

echom 打印之后 用 messages 后会显示，而 echo 不会

## 设置选项

```vim
:set number

:set nonumber

## 切换
:set number!

## 查看值是多少
:set number?
```

## 映射
```vim
:map - x

:map <space> viw

:map <c-d> dd
```




## 参考
1. https://www.w3cschool.cn/vim/nckx1pu0.html



