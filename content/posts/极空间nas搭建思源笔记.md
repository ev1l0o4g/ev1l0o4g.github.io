---
title: "极空间nas搭建思源笔记"
date: 2025-07-17T01:37:02+08:00
draft: false
categories:
  - 杂七杂八
tags:
  - 环境配置
---

## docker下载镜像包

docker仓库搜索`siyuan`，选择第一个下载

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717014935.png)

## 配置容器

通用 -> 取消性能限制

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717015429.png)

文件夹路径 -> 添加极空间路径和装载路径

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717015652.png)

端口 -> 添加登录端口和发布端口

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717020044.png)

环境 -> 添加PUID和PGID

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717020135.png)

命令 -> 设置语言和授权码

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717020409.png)

点击应用即可启动。

## 远程访问

地址端口设置为上面的登录端口，添加快捷方式方便后续使用

![](https://img.ev1l0o4g.xyz/blog-LAS9adkjkfBkjbnbb/20250717020803.png)

这里的远程访问仅限于在其他远程终端安装极空间客户端后在客户端内使用，如需在浏览器中远程访问，需要用VPS做端口映射，或使用其他方式。