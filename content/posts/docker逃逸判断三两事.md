---
title: "Docker逃逸判断三两事"
date: 2023-07-06T14:20:05+08:00
draft: false
categories:
- 内网安全
tags:
- docker
---

最近测试再次遇到一个docker环境的情况，有些东西还是要现查现找，这次顺便再学习学习，一切皆来源于[TWiki](https://wiki.teamssix.com/)。

# 检测判断

## 判断是否为容器环境

```sh
cat /proc/1/cgroup | grep -qi docker && echo "Is Docker" || echo "Not Docker"
```

## 判断逃逸方法

在开始之前对于容器逃逸主要有以下三种方法：
- 不安全的配置
- 相关程序漏洞
- 内核漏洞

> 由于「相关程序漏洞」这种逃逸方法需要根据目标 Docker 的版本去判断，这里暂时没想到从容器内部获取 Docker 版本的方法，因此暂时还不支持这块的检测。

### 不安全的配置
1. 特权模式
```sh
cat /proc/self/status | grep -qi "0000003fffffffff" && echo "Is privileged mode" || echo "Not privileged mode"
```
如果返回 Is privileged mode 则说明当前是特权模式；如果返回 Not privileged mode 则说明当前不是特权模式。
2. 挂载Docker Socket
```sh
ls /var/run/ | grep -qi docker.sock && echo "Docker Socket is mounted." || echo "Docker Socket is not mounted."
```
如果返回 Docker Socket is mounted.说明当前挂载了 Docker Socket；如果返回 Docker Socket is not mounted. 则说明没有挂载。
3. 挂载 procfs
```sh
find / -name core_pattern 2>/dev/null | wc -l | grep -q 2 && echo "Procfs is mounted." || echo "Procfs is not mounted."
```
如果返回 Procfs is mounted. 说明当前挂载了 procfs；如果返回 Procfs is not mounted. 则说明没有挂载。
4. 挂载宿主机根目录
```sh
find / -name passwd 2>/dev/null | grep /etc/passwd | wc -l | grep -q 7 && echo "Root directory is mounted." || echo "Root directory is not mounted."
```
如果返回 Root directory is mounted. 则说明宿主机目录被挂载；如果返回 Root directory is not mounted. 则说明没有挂载。
5. docker remote api未授权访问
```sh
IP=`hostname -i | awk -F. '{print $1 "." $2 "." $3 ".1"}' ` && timeout 3 bash -c "echo >/dev/tcp/$IP/2375" > /dev/null 2>&1 && echo "Docker Remote API Is Enabled." || echo "Docker Remote API is Closed."
```
如果返回 Docker Remote API Is Enabled. 说明目标存在 Docker remote api 未授权访问；如果返回 Docker Remote API is Closed. 则表示目标不存在 Docker remote api 未授权访问。

### 内核漏洞
1. CVE-2016-5195 DirtyCow 逃逸
执行 uname -r 命令，如果在 2.6.22 <= 版本 <= 4.8.3 之间说明可能存在 CVE-2016-5195 DirtyCow 漏洞。
2. CVE-2020-14386
执行 uname -r 命令，如果在 4.6 <= 版本 < 5.9 之间说明可能存在 CVE-2020-14386 漏洞。
3. CVE-2022-0847 DirtyPipe 逃逸
执行 uname -r 命令，如果在 5.8 <= 版本 < 5.10.102 < 版本 < 5.15.25 < 版本 < 5.16.11 之间说明可能存在 CVE-2022-0847 DirtyPipe 漏洞。

## 脚本检测
项目地址：https://github.com/teamssix/container-escape-check
若容器内有wget命令，直接执行：
```sh
wget https://raw.githubusercontent.com/teamssix/container-escape-check/main/container-escape-check.sh -O -| bash
```
若不存在wget命令，可下载到本地再上传到容器执行。