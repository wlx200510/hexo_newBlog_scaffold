---
title: techniques about VsCode
date: 2016-10-25 11:30:45
categories:
- 学习
tags:
- VsCode
- 编译器
- 路径补全
---
记录目前VScode编译器的所有插件和快捷键，方便以后查找
<!-- more -->
## VScode的插件记录
----------
<article>
    <p>用VS code来做网页web开发，目前安装的插件有
    <ul>
        <li>HTML Snippets</li>
        <li>JavaScript Snippet Pack</li>
        <li>jQuery Code Snippets(jQuery提示)</li>
        <li>HTML CSS Support(CSS提示)</li>
        <li>Path Intellisense(HTML中SRC路径自动补全)</li>
        <li>View In Browser(按Ctrl+F1快速在浏览器里浏览)</li>
        <li>HTML CSS Class Completion(写类时的自动提示补全)</li>
        <li>Sass(用scss写css时的语法高亮提示)</li>
        <li>vscode-icons(文件前显示各种图标，一目了然)</li>
        <li>Start git-bash(与git的bash绑定 不常用)</li>
        <li>Syncing 多终端同步VsCode的配置，快捷键等</li>
    </ul>
    使用后发现其实VScode原本就支持Emmet和原生JS方法的各种提示，所以上手比较容易，比ST好的一点就是可以直接跳转到文件或者函数的定义位置.
    </p>
    <p>
        自定义的快捷键代码(文件名keybindings.json)：

        ```json
        [
            {
            "key": "alt+e",
            "command": "cursorEnd",
            "when": "editorTextFocus"
            },
            {
                "key": "alt+d",
                "command": "cursorDown",
                "when": "editorTextFocus"
            },
            {
                "key": "alt+j",
                "command": "scrollLineDown",
                "when": "editorTextFocus"
            },
            {
                "key": "alt+h",
                "command": "scrollLineUp",
                "when": "editorTextFocus"
            },
            {
                "key": "ctrl+d",
                "command": "editor.action.copyLinesDownAction",
                "when": "editorTextFocus && !editorReadonly"
            },
            {
                "key": "alt+l",
                "command": "cursorRight",
                "when": "editorTextFocus"
            },
            {
                "key": "ctrl+y",
                "command": "editor.action.deleteLines",
                "when": "editorTextFocus && !editorReadonly"
            },
            {
                "key": "alt+k",
                "command": "cursorLeft",
                "when": "editorTextFocus"
            }
        ]
        ```

        *在新电脑上配置VS的内容如上，也希望能给他人一个参考，有新发现随时补充。*
    </p>
</article>
