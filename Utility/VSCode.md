# VScode

[TOC]

## 下载
先从官网得到下载地址
https://az764295.vo.msecnd.net/stable/5763d909d5f12fe19f215cbfdd29a91c0fa9208a/VSCode-win32-x64-1.45.1.zip

但是上面的太慢了，可以用下面的CDN加速下载
http://vscode.cdn.azure.cn/stable/VSCode-win32-x64-1.45.1.zip


## 插件
`Ctrl + Shift + P` 输入 `Install Extensions` 进行插件的安装

插件默认安装的位置: `%HOMEPATH%\.vscode\extensions`

常用插件:
- Vim 
- Code Runner
- Markdown Preview Enhanced
- Table Formatter
- Markmap
- vscode-note
- Remote - SSH
- C/C++ Extension Pack
- Python
- One Dark Pro
- eva theme

## 自定义快捷键设置
|            命令            |     绑定键     |         说明          |
| :------------------------- | :------------- | :-------------------- |
| `Terminal: Focus terminal` | ctrl + alt + N | 在代码和终端切换      |
| `Open preview to the side` | alt+m          | 打开 makedown preview |

## 快捷键
一些常用快捷键[^shortcuts]
ctrl + P: 快速打开
ctrl + Shift + P: 打开命令面板
ctrl + Shift + X: 打开扩展
ctrl + `: 打开终端
ctrl + ,: 打开Settings

`ctrl + K` 是一个快捷键前缀


vim插件中，禁用某些快捷键 [^vim_handlekeys], Settings --> vim handle --> Edit in settings.json
```json
{
    "vim.handleKeys": {

        "<C-a>": false,
        "<C-d>": false,
        "<C-s>": false,
        "<C-z>": false
    }
}
```

## 设置
打开文件始终在新标签页打开：`workbench.editor.enablePreview`: `false`
关闭自动更新：`Update: Mode`: `none`
设置 git bash 为默认Shell：
Ctrl + Shift + P: Select Default Profile --> git bash

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

## 调试
参考YouTube[^VSCode_Cool_youtube] 或 B站[^VSCode_Cool_bilibili]的视频

Debug toolbar[^debug_toolbar] 有几个控制程序的执行流程。从左到右分别叫做：continue、step over、step into、step out、restart、stop。


continue：它的作用是立即跳到下个断点
stop：用于立即中断调试
restart：用于立即重启调试

step over和step into 是因为他俩既有相似点也有不同点，关键在于**当前行是否存在函数调用**，
如果存在，那他俩的作用一样：都是**单步执行**，
如果不存在，step over会直接拿到函数的返回结果并运行至下一行，
而step into则会进入当前行函数并运行至该函数的第一行。

step out：它和step into刚好相反，step into是跳入函数，而step out则是跳出函数


在 launch.json 中 request，它有两个值可供选择，一个叫launch，一个叫attach。
使用launch我们可以让VS Code去启动我们的程序，同时启动后的程序还支持调试。
而attach的方式有点不一样，它并不会帮我们启动程序，而是通过为一个**已经在运行且还不支持调试**的程序**注入一个调试器**，让这个程序从不支持调试变成支持调试。
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach by Process ID",
            "processId": "${command:PickProcess}",
            "request": "attach",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "type": "node"
        },

        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\app.js"
        }
    ]
}
```

logpoints: 使用它我们可以以非阻塞、打日志的方式来调试程序
为了**降低普通断点造成请求被阻塞的风险**，我们就可以使用logpoints断点，它以非阻塞、打日志的方式来调试程序。

## snippets
关键字 + Tab ===> 一段代码

以达到快速编写代码，提升开发效率。可以使用[snippet-generator](https://snippet-generator.app/)

`$0`: 光标的位置
`$1`, `$2`: 第1，2个占位

```json
{
	// Place your snippets for c here. Each snippet is defined under a snippet name and has a prefix, body and
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }
	"using stdio": {
		"prefix": "io",
		"body": [
			"#include <stdio.h>",
		],
		"description": "stdio"
	},

	"main function": {
		"prefix": "main",
		"body": [
			"int main() {",
			"    $0",
			"    return 0;",
			"}",
		]
	},

	"print1": {
		"prefix": "p1",
		"body": [
			"printf(\"%${1:You can use d,l,s,c}\", $2);",
		]
	},

	"print2": {
		"prefix": "p2",
		"body": [
			"printf(\"%${1|d,ld,s,c|}\", $2);",
		]
	},
}
```


## 参考
[^shortcuts]: https://cloud.tencent.com/developer/article/2122325
[^VSCode_Cool_youtube]: https://www.youtube.com/watch?v=HfHsX2yxfNg&list=PLNEp_Xli9W_lUQTtRYG7883bGah63UkNB
[^VSCode_Cool_bilibili]: https://space.bilibili.com/30677217/video
[^debug_toolbar]: https://github.com/vscodecool/vscodecool.github.io#23-debug-toolbar%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F
[^vim_handlekeys]: https://blog.csdn.net/weixin_50134791/article/details/121179927

