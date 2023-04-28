# VScode

[TOC]

## 下载
先从官网得到下载地址
https://az764295.vo.msecnd.net/stable/5763d909d5f12fe19f215cbfdd29a91c0fa9208a/VSCode-win32-x64-1.45.1.zip

但是上面的太慢了，可以用下面的CDN加速下载
http://vscode.cdn.azure.cn/stable/VSCode-win32-x64-1.45.1.zip


## 插件
`Ctrl + Shift + P` 输入 `Install Extensions` 进行插件的安装

插件默认安装的位置: `C:\Users\用户名\.vscode\extensions`

常用插件:
- Vim 
- Code Runner
- Markdown Preview Enhanced
- Table Formatter
- Markmap
- vscode-note

## 自定义快捷键设置
|            命令            |     绑定键     |         说明          |
| :------------------------- | :------------- | :-------------------- |
| `Terminal: Focus terminal` | ctrl + alt + N | 在代码和终端切换      |
| `Open preview to the side` | alt+m          | 打开 makedown preview |

## 快捷键
ctrl + P: 快速打开
ctrl + Shift + P: 打开命令面板

## 设置
打开文件始终在新标签页打开：`workbench.editor.enablePreview`: `false`


## 在Windows下创建快捷方式
```bat
mklink /j VSCode VSCode-win32-x64-1.45.1
```

## 字体
在中文字和英文字混合对齐会有问题, 参考这个 [issue](https://github.com/yzhang-gh/vscode-markdown/issues/293)

使用 [更纱黑体](https://github.com/be5invis/Sarasa-Gothic) 可以解决对齐的问题

可以去 https://github.com/be5invis/Sarasa-Gothic/releases/tag/v0.40.4 下载 ttf 并安装。

在VSCode配置一下字体，并重启一下VSCode。
Font Family: "Sarasa Mono Slab SC"


## 添加右键 vscode 打开
1. 打开注册表: `Win + R` --> `regedit`
2. 定位到: HKEY_CLASSES_ROOT\*\shell 非 administrator `Computer\HKEY_CURRENT_USER\SOFTWARE\Classes\*`
3. 新建 key: Open with VSCode
4. 在“Open with VSCode”上再新建key：command
5. 修改Value Data: `C:\Users\bzhux\Program\VSCode\Code.exe %1`


