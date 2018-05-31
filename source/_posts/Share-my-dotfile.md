---
title: Share my dotfile
copyright: true
date: 2018-05-31 16:06:35
tags: [vim, zsh, terminal]
comments:
---

#### 工作环境推荐
笔者的工作环境是`Ubuntu-16.04.1 + Neovim + zsh + oh-my-zsh + tmux + oh-my-tmux`，使用到图形界面的时候主要是阅读 pdf 文档以及浏览网页。

身为一个重度 Vim 使用者，我交换了 ESC 键和 Caps Lock 键，使用 zsh 的 `Vi` 模式，并尝试在一切可能的地方使用`Vim-like`的方式来操控机器。

* 网页浏览：Chrome + cVim(Chrome 插件）
* PDF 阅读： Evince（测试过所有 `Vim-like` 的阅读器之后，发现或多或少存在问题，Evince 只能算是差强人意的选择）
* PDF 制作： Neovim + Typora
* PPT 制作： Neovim + nodePPT
* Overwall： VPS + Shadowsocks
* 文件管理： VPS + Git（这里强烈批评坚果云，居然能 24 小时吃掉我 100% 的 CPU 资源，对富文本格式文件解析能力非常差）
* 文件共享： VPS + Git
* 终端模拟器： Tilix
* 键鼠 + 剪贴板共享： Synergy

折腾过很久的 Vim 插件，所走的这些路，应该是大多数被 Vim 哲学所吸引的人都走过的。
在这里分享 dotfile，以飨后来人。希望或多或少能帮到你。

<!--more-->

> https://github.com/DarcySail/dotfile

本仓库会持续更新，除了去掉敏感的隐私内容之外，会尽量与我实际使用的 dotfile 保持一致。

#### To be tested
* 文件浏览： 的确有 `Vim-like` 的文件系统管理器，但我对此心存疑虑，zsh 组合拳似乎对我已经非常够用了

#### 未来目标
* 将 dotfile 中的所有英文整理为中文（本博客的定位是创造价值以及帮助后来者，所以会照顾大多数 newcomer）；
* 测试完所有 Github 上面排名前十的 dotfile 仓库，整合到我自己的内容中来；
* 排版格式
