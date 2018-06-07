---
title: Build OpenVPN Server on 53 port - The Easiest Way
copyright: true
date: 2018-06-06 23:19:59
tags: [OpenVPN, crack]
comments:
---

#### 序言
本文旨在教会对 Linux 稍有基础的同学，以__最简单__的方式在拥有__公网 IP__ Ubuntu
主机上搭建一个监听 __UDP 53 端口__ __OpenVPN__ 服务器，并配置好 Linux
client 端，以实现绕过__Portal 认证__的目的。
> 加黑的每个关键词要么可以搜出大量相关资料，要么是接下来的步骤的关键。
> 本文不会告诉你为什么要这么做，上面给出的关键词已经是充足的暗示。

#### 正文
1. 使用 ssh 登录你的 Ubuntu 服务器，在命令行中输入：`netstat -anup | grep 53`
```bash
udp        0      0 0.0.0.0:5355            0.0.0.0:*                           446/systemd-resolve
udp        0      0 127.0.0.1:53            0.0.0.0:*                           543/dnsmasq
udp        0      0 127.0.0.53:53           0.0.0.0:*                           446/systemd-resolve
.
.
udp6       0      0 :::5355                 :::*                                446/systemd-resolve
udp6       0      0 ::1:53                  :::*                                543/dnsmasq
```
    倘若上面这条命令输出大量结果，请先暂时离开这篇文章，去调研如何先关闭你机器上监听 53 端口的服务。因为倘若有别的服务占用了 53 端口，会与我们接下来的步骤产生冲突。
    倘若没有任何结果输入，恭喜，进入下一步骤。
2. 继续在命令行中输入：
    ```bash
    wget https://git.io/vpn -O openvpn-install.sh && bash openvpn-install.sh
    ```
    > 参考资料：https://github.com/Nyr/openvpn-install

    得到如下结果：

    ```bash
    Welcome to this OpenVPN "road warrior" installer!
    
    I need to ask you a few questions before starting the setup.
    You can leave the default options and just press enter if you are ok with them.
    
    First, provide the IPv4 address of the network interface you want OpenVPN
    listening to.
    IP address: XX.XX.XX.XX #此处隐匿的是笔者机器的公网 IP，不用做任何修改，直接回车

    Which protocol do you want for OpenVPN connections?
       1) UDP (recommended)
       2) TCP
    Protocol [1-2]: 1
    
    What port do you want OpenVPN listening to?
    Port: 53 #注意此处一定要修改为 53，这是最为关键的一步
    
    Which DNS do you want to use with the VPN?
       1) Current system resolvers
       2) 1.1.1.1
       3) Google
       4) OpenDNS
       5) Verisign
    DNS [1-5]: 3 #此处修改为 3
    
    Finally, tell me your name for the client certificate.
    Please, use one word only, no special characters.
    Client name: client
    
    Okay, that was all I needed. We are ready to set up your OpenVPN server now.
    Press any key to continue...
    ```
3. 此时耐心等待脚本安装软件包，以及配置服务器 config 文件。根据机器性能不同，等待时间从几十秒到十几分钟不等。所幸不用任何操作，可以去喝杯咖啡再回来。
待到看到如下字样时，OpenVPN 安装并配置成功，而且已经默认跑起来了。
```bash
Finished!

Your client configuration is available at: /root/client.ovpn
If you want to add more clients, you simply need to run this script again!
```

4.  注意输出的这一条语句：__/root/client.ovpn__，使用`scp`将改文件从服务器拷贝到你的 client 端机器上。
由于每个人登录的账户不同，具体的输出路径可能不一样。
```bash
scp username@XX.XX.XX.XX:yourfilepath .
```

5. 打开你的 Ubuntu client 机器上的网络管理，选择`import a saved VPN configuration`，选中拷贝下来的__client.ovpn__文件，在弹出窗口中不用做任何修改，直接点击 saved 保存即可。

#### 撤销操作
如果想要撤销服务器上的操作，再次运行下面这条指令。选择__3__即可。

```bash
wget https://git.io/vpn -O openvpn-install.sh && bash openvpn-install.sh

Looks like OpenVPN is already installed.

What do you want to do?
    1) Add a new user
    2) Revoke an existing user
    3) Remove OpenVPN
    4) Exit
Select an option [1-4]: 3

Do you really want to remove OpenVPN? [y/N]: y
```
