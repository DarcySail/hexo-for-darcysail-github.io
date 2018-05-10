---
title: 关闭 tmux panel 无需确认
date: 2018-05-10 15:46:40
tags: [tmux]
comments: tmux tips
copyright: true
---

使用 tmux 时经常开启关闭临时面板（panel），每次关闭时 tmux 都会让你确认是否真的要关闭，有些繁琐。
将如下配置加入你的`.tmux.conf`文件来避免繁琐的确认步骤：

```
bind-key & kill-window
bind-key x kill-pane
```
重启 tmux 服务，或者重新加载 tmux 配置文件（`.tmux.conf`）后生效。

参考链接：
https://unix.stackexchange.com/questions/30270/tmux-disable-confirmation-prompt-on-kill-window

<!--more-->

回答部分翻译如下：

前缀键（通常指 ctrl+b）& 通常与`confirm-before -p "kill-window #W? (y/n)" kill-window`相绑定，`confirm-before`导致你需要进行确认的操作。如果你想避免确认步骤，只需要将原有的按键命令解绑，然后直接将其与`kill-window`重新绑定即可。

如果你想要知道你的 tmux 配置默认有哪些操作需要确认步骤，使用如下命令：
```
tmux list-keys | grep confirm-before
```