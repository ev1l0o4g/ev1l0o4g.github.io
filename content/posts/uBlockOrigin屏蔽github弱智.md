---
title: "UBlockOrigin屏蔽github弱智"
date: 2023-11-30T10:41:25+08:00
draft: false
categories:
- 杂七杂八
tags:
- 插件
---

uBlock Origin插件在自定义静态规则添加如下规则：
```
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(cirosantilli)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(cheezcharmer)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(zaohmeing)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(zhaohmng-outlook-com)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(codin-stuffs)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(Dimples1337)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(pxvr-official)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(zpc1314521)
github.com##.Box-sc-g0xbh4-0.hKtuLA:has-text(b0LBwZ7r5HOeh6CBMuQIhVu3-s-random-fork)
```

![](https://s2.loli.net/2023/11/30/FYRb73qrCjA8fVy.png)

> 2025/3/10更

其实油猴插件有相关脚本，比如：

[GitHub 去除中文搜索污染](https://greasyfork.org/zh-CN/scripts/470797-github-%E5%8E%BB%E9%99%A4%E4%B8%AD%E6%96%87%E6%90%9C%E7%B4%A2%E6%B1%A1%E6%9F%93)

[Github搜索净化](https://greasyfork.org/zh-CN/scripts/473912-github%E6%90%9C%E7%B4%A2%E5%87%80%E5%8C%96)