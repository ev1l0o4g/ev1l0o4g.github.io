---
title: "记win11终端ssh配置和Ubuntu设置root登录"
date: 2023-11-12T20:21:17+08:00
draft: false
categories:
- 杂七杂八
tags:
- 环境配置
---

# win11终端配置

win11自带SSH，如果在把服务器添加到win11终端岂不是方便了许多。
操作很简单，在终端新建配置文件，命令行改为ssh的命令，给配置文件起个名，保存就完事
![nzvhnv.png](https://files.catbox.moe/nzvhnv.png)
甚至还能设置自己心爱的logo，缺点就是每次要输密码，不过也可以密钥免密登录
![gx92ai.png](https://files.catbox.moe/gx92ai.png)

# Ubuntu设置root登录

上次出差因为酒店房间线路短路，导致前任电脑出现了硬伤，无奈只好含（喜）泪重（提）金购置小“外星人”。
重装ubuntu虚拟机就很烦，桌面版ubuntu默认禁止root登录，想要以root用户进行登录需要进行一些操作，如下：
```

一、
修改文件/usr/share/lightdm/lightdm.conf.d/50-unity-greeter.conf文件，增加两行： 
greeter-show-manual-login=true 
all-guest=false 
保存

二、
进入/etc/pam.d目录，修改gdm-autologin和gdm-password文件 
vi gdm-autologin 
注释掉auth required pam_succeed_if.so user != root quiet_success这一行，保存 
vi gdm-password 
注释掉 auth required pam_succeed_if.so user != root quiet_success这一行，保存

三、
修改/root/.profile文件 
vi /root/.profile 
将文件末尾的mesg n || true这一行修改成tty -s&&mesg n || true， 保存

四、
重启系统，输入root用户名和密码，登录系统。

五、
vim /etc/gdm3/custom.conf
将[daemon]
AutomaticLoginEnable=True
AutomaticLogin=root

```

参考自https://blog.csdn.net/GoodFaith008/article/details/132041763