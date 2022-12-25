---
title: LFS 部分问题的解决方案
copyright: true
date: 2018-08-15 16:04:18
tags: [Linux, LFS]
comments:
---

本文记录一些在尝试 LFS 8.2 时遇到的一些问题的解决方案。
这些解决方案被我选录的标准是，英文网络中有，但中文网络目前还没有。以飨后来人。

--------------------------------------------------------------------------------

1. 在为 LFS 编译 Glibc 库的时候，会在 `./configure`期间遇到如下问题：

```bash
LD_LIBRARY_PATH shouldn't contain the current directory
checking LD_LIBRARY_PATH variable... contains current directory
configure: error:
*** LD_LIBRARY_PATH shouldn't contain the current directory when
*** building glibc. Please change the environment variable
*** and run configure again.
```

出现这个错误的原因是由于环境变量的 `LD_LIBRARY_PATH` 中出现了当前目录。通常我们修改环境变量时会这么写：

```bash
export LD_LIBRARY_PATH = $LD_LIBRARY_PATH:/foo/bar:/hello/world:/a/b
```

如果一开始 `LD_LIBRARY_PATH` 不存在的话，这个上面这串环境变量开头就是冒号，这就把当前文件夹包含进去了。一般来说我们挺需要这种效果，因为在编译的时候可以 include 某些东西，但是对于编译 glibc 来说这个是多余的。

<!--more-->

最简单的解决方法就是 `unset LD_LIBRARY_PATH`。但这个解决方案会在接下来 LFS 的一系列步骤中，产生一些问题。

> 1. https://github.com/Linuxbrew/legacy-linuxbrew/issues/807
> 2. `LIBRARY_PATH` 用于在链接过程中查找库。`LD_LIBRARY_PATH` 用于在程序执行期间查找库。

--------------------------------------------------------------------------------
2. 在为 LFS 第二遍编译 Binutils-2.25 的时候，会在 `./configure`期间遇到如下问题：

```bash
no such file or directory: ./a.out
```

日志如下（注，这段日志并不重要，只是为了增加被搜索引擎索引的准确率，方便后来人查找到这一问题）：
```bash
checking for C compiler default output file name... a.out
checking whether the C compiler works... configure: error: in `/mnt/sources/binutils-build':
configure: error: cannot run C compiled programs.
If you meant to cross compile, use `--host'.
See `config.log' for more details.
```

My problem is glibc was in /lib and /lib64 was not a link to /lib. My fix was mv /tools/lib64/* /tools/lib/ && rmdir /tools/lib64 && ln -s lib /tools/lib64
翻译：我遇到的问题在于 glic 位于`/lib`路径下，而`/lib64`并没有链接（注，指 ln 命令的软链接）到`/lib`目录。我解决这个问题的办法是运行下面这条命令：
```bash
mv /tools/lib64/* /tools/lib/ && rmdir /tools/lib64 && ln -s lib /tools/lib64
```

> 参考链接：https://www.reddit.com/r/linuxfromscratch/comments/2yd1j5/59_binutils225_pass_2_failing_during_configure/

--------------------------------------------------------------------------------

3. LFS 书中最坑的一处小细节是，在 `chroot` 之后，它完成 `/mnt/lfs/boot/` 到宿主`/boot`路径的绑定了，就直接让读者对 `/boot/grub/grub.cfg`文件进行修改。 此刻你可能还沉浸在 `chroot` 带给你的安全感里，一不留神就中招。

重启并发现 grub 引导失败之后，大多数用户应该使用的 grub2 而非 grub。这两个程序重新进入引导的方式是不一样的。
简单记录 grub2 手动重新引导过程如下，__注意，不要直接输入下面的代码！__：
```bash
grub> set root=(hd0,1)
grub> linux /boot/vmlinuz-3.13.0-29-generic root=/dev/sda1
grub> initrd /boot/initrd.img-3.13.0-29-generic
grub> boot
```

将第一行中的 `(hd0,1)` 替换为你原本 `/mnt/lfs` （也就是 `chroot` 所用到的地址）所挂在的硬盘地址。这里值得一提的是，grub 对于硬盘命名的方式有细微差别，一定要弄清之后再根据你机器的实际情况做修改。
将第二行和第三行的路径都需要替换为你编译好的内核路径，第二行中的root路径与第一行的路径相同（只是写法不一样）。

> 参考链接：https://www.linux.com/learn/how-rescue-non-booting-grub-2-linux
