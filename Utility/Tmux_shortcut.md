# Tmux快捷键

[TOC]

[转载于这里](http://www.linuxidc.com/Linux/2015-07/119843.htm)

## tmux的基本概念
### Session 
    一组窗口的集合，通常用来概括同一个任务。session可以有自己的名字便于任务之间的切换。
### Window 
    单个可见窗口。Windows有自己的编号，也可以认为和ITerm2中的Tab类似。
### Pane 
    窗格，被划分成小块的窗口，类似于Vim中 C-w +v 后的效果。
![0](images/tmux/0.jpg)


## 基本操作

所有快捷键的执行方式:

按下control + b两个按键组合, 然后松开control + b(为了告诉Tmux我要用Tmux的快捷键了), 然后在按快捷键触发各种行为。

例如: Ctrl + B + ?的执行过程为按下control + b两个按键组合, 然后松开control + b, 然后在按’?’键, 会显示所有快捷键的列表。

Ctrl + B + ? 列出所有快捷键, 按q或Esc返回
Ctrl + B + d detach当前会话,可暂时返回Shell界面，输入tmux attach能够重新进入之前会话
Ctrl + B + s 选择并切换会话；在同时开启了多个会话时使用

## 快捷键

```
面板操作    ”   将当前面板平分为上下两块
%   将当前面板平分为左右两块
x   关闭当前面板
!   将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板
Ctrl+方向键    以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键 以5个单元格为单位移动边缘以调整当前面板大小
Space   在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled
q   显示面板编号
o   在当前窗口中选择下一面板
方向键 移动光标以选择面板
{   向前置换当前面板
}   向后置换当前面板
Alt+o   逆时针旋转当前窗口的面板
Ctrl+o  顺时针旋转当前窗口的面板
```

## Window操作
|快捷键|说明|
|:-----------|:----------|
|Ctrl + B + , | 重命名当前窗口，便于识别各个窗口
|Ctrl + B + . | 修改当前窗口编号；相当于窗口重新排序
|Ctrl + B + f | 在所有窗口中查找指定文本

## Window操作
以 ***Ctrl + B*** 为前缀

| 快捷键       | 说明                          |
| :----------- | :----------                   |
| c            | 新增一个window                |
| &            | 退出当前window                |
| ,            | 重命名当前window              |
| l            | 前后两个window之间切换        |
| i            | 显示当前window的信息          |
| w            | 列出所有的window选择          |
| 0 to 9       | 切换window 到相应编号的window |
| p            | 切换window 上一个window       |
| n            | 切换window 下一个window       |
| '            | 切换window 到输入编号的window |
| f            | 切换window 到搜索到的window   |
| Space        | 改变当前window下的pane布局    |



## Pane操作 
以 ***Ctrl + B*** 为前缀

| 快捷键               | 说明                                    |
| :-----------         | :----------                             |
| ！                   | 从window移除当前pane                    |
| "                    | 将当前pane变成上下两个pane              |
| %                    | 将当前pane变成左右两个pane              |
| x                    | 关闭当前pane                            |
| q                    | 显示pane的索引                          |
| z                    | 最大化或者恢复当前pane                  |
| {                    | 跟前一个pane交换位置                    |
| }                    | 跟后一个pane交换位置                    |
| o                    | 切换Pane 到下一个pane                   |
| ;                    | 切换Pane 进入到前一个操作过的pane       |
| Up, Down Left, Right | 切换Pane 使用方向键切换到相应方向的pane |
| [ + 上下键           | 滚屏                                    |


## Session操作
以 ***Ctrl + B*** 为前缀

| 快捷键       | 说明                                             |
| :----------- | :----------                                      |
| Ctrl + z     | 关闭tmux                                         |
| :            | 进入tmux命令行模式                               |
| ?            | 列出所有快捷键                                   |
| t            | 显示时间                                         |
| d            | 退出当前tmux客户端，tmux后台运行                 |
| $            | 重命名当前session                                |
| s            | 切换session 显示所有session并切换到某一个session |
| (            | 切换session 切换到上一个session                  |
| )            | 切换session 切换到下一个session                  |
| L            | 切换session 到前一个活跃的session                |


创建一个新的session
tmux new-s [name-of-my-session]

在当前session中创建一个新的Session, 并保证之前session依然存在
Ctrl + B + new -s [name-of-my-new-session]

|快捷键|说明|
|:-----------|:----------|
|tmux | 开启tmux
|tmux ls | 显示已有tmux列表 (等同于Ctrl + B + S)
|tmux attach -t test | 进入名为test的session
|Ctrl + B + s | 选择并切换会话；在同时开启了多个会话时使用
|Ctrl + B + d | 脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话
|Ctrl + B + D | detach当前session(可以认为后台运行)
|Ctrl + B + r | 强制重绘未脱离的会话
|Ctrl + B + : | 进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器
|Ctrl + B + ~ | 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息
|Ctrl + z | 挂起当前会话

| 快捷键                      | 说明                                                    |
| :-----------                | :----------                                             |
| tmux new -s session_name    | creates a new tmux session named session_name           |
| tmux attach -t session_name | attaches to an existing tmux session named session_name |
| tmux switch -t session_name | switches to an existing session named session_name      |
| tmux list-sessions          | lists existing tmux sessions                            |
| tmux new-window             | create a new window                                     |
| tmux select-window -t :0-9  | move to the window based on index                       |
| tmux rename-window          | rename the current window                               |
|                             |                                                         |

## 进阶
美化Tmux

使用gpakosz的Tmux配置进行美化。

优点

使用Ctrl + a作为前缀更方便使用, 同时保存了Ctrl + B +的触发前缀
powerline状态条美化(用过vim的都应该比较熟悉)
显示笔记本电池状态

安装使用
```bash
$ cd
$ rm -rf .tmux
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local.
```

## 复制与粘贴
复制：`Ctrl + B + [` 进入 visual 模式,  到你要复制的内容的开始处，按 空格键 开始复制, 然后移动到要复制内容的最尾端， 按 回车键 完成复制。

粘贴: `Ctrl + B + ]`

tmux 1.8 发布,Linux 终端复用器 http://www.linuxidc.com/Linux/2013-03/81980.htm
Tmux：终端复用器 http://www.linuxidc.com/Linux/2013-07/86776.htm
tmux使用简单教程 http://www.linuxidc.com/Linux/2014-10/107644.htm
CentOS下Tmux安装和使用 http://www.linuxidc.com/Linux/2014-11/109375.htm
用 Tmux 和 Vim 打造 IDE  http://www.linuxidc.com/Linux/2015-06/119165.htm
本文永久更新链接地址：http://www.linuxidc.com/Linux/2015-07/119843.htm

