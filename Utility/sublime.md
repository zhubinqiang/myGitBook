# sublime
[TOC]

## 快捷键
### 编辑
### 编辑
- `Ctrl + X` 删除行
- `Ctrl + Shift + D` 复制当前行
- `Ctrl + Shift + 上下键` 该行上下移动
- `Ctrl + /` 注释当前行
- `Alt + Shift + 1/2/3/4` 分屏显示
- `Alt + 数字` 切换第N个页签
- `Ctrl + Alt + F` 格式化代码
- `Ctrl + Alt + 上下键` 选中多行 在各行的光标处编辑

### 选择
- `Ctrl + D` 选中光标所占文本， 继续操作会选中下一个相同的文本
- `Alt + F3` 选中文本进行同时编辑， 比如快速修改变量名
- `Ctrl + Shift +[` 先选中代码，执行此命令，折叠代码
- `Ctrl + Shift +]` 先选中代码，执行此命令，展开代码
- `Ctrl + K + 0` 展开所有折叠代码
- `Shift + 上下键` 选中多行

### 搜索
- `Ctrl + F` 打开底部搜索框，查找关键字 
- `Ctrl + Shift + F` 在文件夹内查找 
- `Ctrl + H` 替换 

- `Ctrl + P`  打开顶部搜索框
    + 文件名： 搜文件
    + `@ + 关键字`: 搜索函数名
    + `# + 关键字`: 搜索变量名
    + `: + 数字`： 跳转到该行
- `Ctrl + R` 打开顶部搜索框，自动带@。 搜索函数名
- `Ctrl + :` 打开顶部搜索框，自动带#。 搜索变量名，属性


## Settings
```json
{
    "color_scheme": "Packages/Color Scheme - Default/All Hallow's Eve.tmTheme",
    "font_face": "YaHei Consolas hybrid",
    "font_size": 14,
    "highligt_line": true,
    "ignored_packages":
    [
    ],
    "theme": "Spacegray Light.sublime-theme",
    "update_check": false
}
```


## Package Control
```json
{
    "bootstrapped": true,
    "http_proxy": "www.example.com:913",
    "https_proxy": "www.example.com:913",
    "in_process_packages":
    [
    ],
    "installed_packages":
    [
        "1337 Color Scheme",
        "CodeFormatter",
        "ConvertToUTF8",
        "GBK Support",
        "Git",
        "Groovy Snippets",
        "IMESupport",
        "jQuery",
        "Markdown Preview",
        "MarkdownEditing",
        "Nodejs",
        "Notes",
        "Package Control",
        "PlainNotes",
        "SublimeREPL",
        "Table Editor",
        "View In Browser"
    ]
}
```

## Markdown Preview
### settings
```json
{
//  "enabled_parsers": ["markdown", "wikilinks"] 

    "parser": "markdown",
    "build_action": "browser",
    "enable_mathjax": true,
    "enable_uml": true,
    "enable_highlight": true,
    "enable_pygments": true,
    "css": ["C:\\Users\\bzhux\\Program\\SublimeTextBuild3126x64\\Data\\myconf\\github.css"],
    // "enabled_parsers": ["markdown", "github"],
    "enabled_parsers": ["markdown"],
    "github_mode": "markdown",
    "github_inject_header_ids": true,
    "enable_autoreload": false
}
```



## MarkdownEditing
### settings for gfm
```json
{
    "enable_table_editor": true,
    "extensions":
    [
        "md"
    ],
    "highlight_line": true,
    "line_numbers": true
}

```

## PlainNotes
### settings
```json
{
    "root": "~/WS/Notes/",
    "note_color_scheme": "Packages/PlainNotes/Color Schemes/Sticky-Gray.tmTheme",
    "jotter_color_scheme": "Packages/PlainNotes/Color Schemes/Sticky-Gray.tmTheme",
    "list_options" : {
    "display_modified_date": true,
    "display_folder": true,
    "display_full_path": true
  }
}
```

## Nodejs
### settings
```js
{
  // save before running commands
  "save_first": true,
  // if present, use this command instead of plain "node"
  // e.g. "/usr/bin/node" or "C:\bin\node.exe"
  "node_command": "C:\\Program Files\\nodejs\\node.exe",
  // Same for NPM command
  "npm_command": "C:\\Program Files\\nodejs\\npm.cmd",
  // as 'NODE_PATH' environment variable for node runtime
  "node_path": false,

  "expert_mode": false,

  "ouput_to_new_tab": false
}
```


## Key
```json
[{
    "keys": ["alt+right"],
    "command": "next_view"
}, {
    "keys": ["alt+left"],
    "command": "prev_view"
}, {
    "keys": ["alt+m"],
    "command": "markdown_preview",
    "args": {
        "target": "browser"
    }
}, {
    "keys": ["ctrl+alt+f"],
    "command": "reindent",
    "args": {
        "single_line": false
    }
}, {
    "keys": ["ctrl+alt+enter"],
    "command": "open_in_browser"
}, {
    "keys": ["f5"],
    "caption": "SublimeREPL: Python - RUN current file",
    "command": "run_existing_window_command",
    "args": {
        "id": "repl_python_run",
        "file": "config/Python/Main.sublime-menu"
    }
}]
```

## Ctrl + B
Tools --> Build System --> New Build System
new Groovy.sublime-build
### Groovy
```
{
    "cmd": ["groovy","$file"],
    "selector": "source.groovy",
    "file_regex": "[ ]*at .+[(](.+):([0-9]+)[)]",

    "windows": {
        "shell": "cmd.exe"
    }
}
```

