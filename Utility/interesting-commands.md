# 有意思的命令

## procs
[procs](https://github.com/dalance/procs) 是一个用rust写的来代替 `ps` 命令。

```bash
sudo snap install procs
```

## motd
一些有意思的 [motd](https://github.com/abcfy2/motd)。 message of the day
需要把内容放到 `/etc/motd`。 ssh 登录的时候就能看到 显示信息了

## cowsay
cowsay
```bash
sudo apt install -y cowsay
cowsay "Hello!"
```

类似的还有 boxes
```bash
sudo apt install -y boxes
echo "123" | boxes -d dog
```

figlet 与 toilet 能显示大号的ASCII字符
```bash
sudo apt install -y figlet toilet
figlet A B C
toilet A B C
```

## lolcat
[lolcat](https://github.com/tehmaze/lolcat.git) 提供彩虹色
```bash
pip3 install lolcat

echo A B C | lolcat
```

## fortune
随机打印格言
```bash
sudo apt install -y fortune
```

## tldr
[tldr](https://github.com/tldr-pages/tldr) 是 Too Long; Didn't Read. 提供了更友好的 man pages.
```bash
pip3 install tldr

tldr tmux
```

## bat
[bat](https://github.com/sharkdp/bat) 是`cat`的增强版。对编程语言提供了语法高亮。
```bash
sudo apt install -y bat
ln -s `which batcat` ~/.local/bin/bat

batcat README.md
```
> Ubuntu下，它叫batcat。可以为它做一个bat的软连接。

## htop
[htop](https://github.com/htop-dev/htop) 是一个 `top` 的替代品

## diff-so-fancy
[diff-so-fancy](https://github.com/so-fancy/diff-so-fancy/releases/) 一个更好的diff工具

在 git 里面需要配置一下
```bash
git config --global core.pager "diff-so-fancy | less --tabs=4 -RF"
git config --global interactive.diffFilter "diff-so-fancy --patch"
git config --global color.ui true
git config --global color.diff-highlight.oldNormal    "red bold"
git config --global color.diff-highlight.oldHighlight "red bold 52"
git config --global color.diff-highlight.newNormal    "green bold"
git config --global color.diff-highlight.newHighlight "green bold 22"
git config --global color.diff.meta       "11"
git config --global color.diff.frag       "magenta bold"
git config --global color.diff.func       "146 bold"
git config --global color.diff.commit     "yellow bold"
git config --global color.diff.old        "red bold"
git config --global color.diff.new        "green bold"
git config --global color.diff.whitespace "red reverse"
```

## ncdu
[ncdu](https://dev.yorhel.nl/ncdu) 是 `du` 的替代品。
```bash
sudo apt install -y ncdu
```

## ack
[ack](https://beyondgrep.com) 是 `grep` 的替代品。

```bash
sudo apt install -y ack
```

ack 默认不支持markdown，需要在 `~/.ackrc` 配置一下[^ack]
```bashrc
--type-set=md=.md,.mkd,.markdown
--pager=less -FRX
```

这个示例是在 md 文件中, 寻找 “git clone” 字符串。
```bash
ack "git clone" --md
```


## 引用
[^ack]: https://zhuanlan.zhihu.com/p/48076652


