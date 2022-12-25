---
title: Vim 中文乱码的非典型解决方案
copyright: true
date: 2018-08-20 14:01:41
tags: [Vim]
comments:
---

#### TLDR
1. 在.vimrc 或者你的相应 Vim 配置文件中删去`set binanry`
2. 根据你打开的文件编码（如何查询文件格式编码，可以使用`enca -L chinese filename`），在 Vim 中输入，譬如`:edit ++enc=gb18030`

#### 正文

前些天从用了一年多了 Linux Mint 转投 Arch Linux 门下，然后在Vim里编辑网络上下载下来的字幕文件时，发现了中文乱码的现象。当时心态有点糟，用了这么久的Vim，居然阴沟翻船。
我的环境是 Arch + NeoVim，配置文件是从我自己用了很久的没出问题的 .vimrc 拷贝过来的。

尝试了网上种种乱码解决方案无效之后，决定自行解决问题。

<!--more-->

#### Solution 1
大部分会要求你在 .vimrc中进行如下配置[[1]](#1)：
```vim
set encoding=utf-8
set termencoding=utf-8
set fileencodings=utf-8,gbk,latin1
```
* <span id = "1"> [1] https://blog.csdn.net/smstong/article/details/51279810</span>


这些配置，我当然是有的……扶额……

经检查发现是 .vimrc中的`set binary`在作祟。输入`:help binary`查看说明：
> The 'fileencoding' and 'fileencodings' options will not be used, the file is read without conversion.

binay选项会导致fileencoding 和fileencodings失效，这就是在进行网上大部分的配置之后，仍会失效的原因。

> https://superuser.com/questions/663405/consequences-of-vims-binary-mode

#### Solution 2
除了取消binary选项之外，也可以在打开的文件中输入如下命令：（具体参数根据文件编码格式而异）
```
:edit ++enc=gb18030
```

> https://blog.csdn.net/cnwzh/article/details/7024427

#### 题外话
关于我为什么要配置`set binary`选项，请参看stack overflow问答
[Why do I need vim in binary mode for 'noeol' to work?](https://stackoverflow.com/questions/16222530/why-do-i-need-vim-in-binary-mode-for-noeol-to-work)
