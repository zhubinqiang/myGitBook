# vim 插件管理利器 Vundle
[TOC]

## 安装
[Vundle](https://github.com/VundleVim/Vundle.vim)是一个vim 插件管理利器

```sh
mkdir -p ~/.vim/bundle
cd ~/.vim/bundle/
git clone https://github.com/VundleVim/Vundle.vim.git
```

## 配置 vim
放在 `~/.vimrc` 的最上面.
```vim
set nocompatible               " be iMproved
filetype off                   " required!
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'


call vundle#end()
filetype plugin indent on
```

上面空白处填写插件的配置 比如
```vim
set nocompatible               " be iMproved
filetype off                   " required!
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'

Plugin 'tpope/vim-sensible'
Plugin 'vim-scripts/UltiSnips'

Plugin 'groovy.vim'

call vundle#end()
filetype plugin indent on
```

1. 这里的 'tpope/vim-sensible' 它会到github上 'https://github.com/tpope/vim-sensible' 搜寻。 
2. 'groovy.vim' 会到 https://www.vim.org/scripts/ 上搜寻。

## Vundle 插件命令
1. `:PluginList`: lists configured plugins
2. `:PluginInstall`: installs plugins; append `!` to update or just `:PluginUpdate`
3. `:PluginSearch foo`: searches for foo; append `!` to refresh local cache
4. `:PluginClean`: confirms removal of unused plugins; append `!` to auto-approve removal

## 安装 markdown插件
1. 追加到 `~/.vimrc`
```vim
Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'
let g:vim_markdown_folding_disabled = 1
" set conceallevel=2
let g:vim_markdown_emphasis_multiline = 0
let g:vim_markdown_frontmatter = 1
let g:vim_markdown_new_list_item_indent = 2
```

2. 启动 `vim` 运行 `run :PluginInstall`
