---
title: "记灯塔ARL的搭建"
date: 2023-11-12T22:03:25+08:00
draft: false
categories:
- 杂七杂八
tags:
- 环境配置
---

灯塔找资产还是挺好用的！

# docker安装ARL

```shell
cd /opt/
mkdir docker_arl
wget -O docker_arl/docker.zip https://github.com/TophantTechnology/ARL/releases/download/v2.6/docker.zip
cd docker_arl
unzip -o docker.zip
docker-compose pull
docker volume create arl_db
docker-compose up -d
```
如果pull失败，要么用代理，要么换docker镜像源。记得docker启动前修改配置文件，添加fofa/hunter/quake密钥。

# 批量导入POC

在docker_arl目录创建poc文件夹，将POC文件放在/docker_arl/poc路径，docker重启ARL即可。

# 批量导入指纹

要想批量导入指纹，可以使用使用[ARL-Finger-ADD](https://github.com/loecho-sec/ARL-Finger-ADD)
ARL-Finger-ADD批量导入的是Ehole3.0的指纹，嫌指纹少可以用各大佬的魔改版Ehole指纹。