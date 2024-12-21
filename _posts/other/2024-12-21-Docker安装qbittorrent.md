---
title: "Docker 安装 qBittorrent"
subtitle: "Docker | qBittorrent | 下载器 ｜ 认证问题"
layout: post
author: "luoruiqing"
header-style: text
catalog:    true
tags:
  - Docker
---



Docker中安装qBittorrent下载器的问题

最近购置了MacMini M4， 选择下载器的时候发现迅雷某些资源还是受限，有版权要求的情况，网上对比下来qBittorrent是比较合适的下载器，正好Docker中可以进行搭建


## 启动

启动参数如下， 注意对应的卷进行挂载目录，用于配置留存到本地，当容器删除后依然可以再次创建容器并继续任务。


```shell
docker run -d \
  --name=qbittorrent \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e WEBUI_PORT=8080 \
  -e TORRENTING_PORT=6881 \
  -p 8080:8080 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v <你的本机目录>:/config \
  -v <你的本机目录>:/downloads \
  --restart unless-stopped \
  lscr.io/linuxserver/qbittorrent:latest

```


## 未认证

如果提示 `unauthorized` 的情况， 则是因为开启了认证，根据如下顺序进行操作

1. 至少启动一次容器
2. 停止容器
3. 找到本地目录的映射卷地址，例如 `/Users/xxx/docker/qbittorrent/appdata/qBittorrent`目录
4. 修改文件 `qBittorrent.conf` 增加 `WebUI\HostHeaderValidation=false`
5. 重启容器


## 容器自动重启

在Mac中安装的Docker并且用配置界面启动的容器，有些参数无法指定，此时可以通过命令行对已启动的容器进行参数的变更


```bash
# 查看容器列表
docker ps
# 如下
CONTAINER ID   IMAGE                            COMMAND   CREATED          STATUS          PORTS                                                                     NAMES
2d1d22f00ddb   linuxserver/qbittorrent:latest   "/init"   51 minutes ago   Up 49 minutes   0.0.0.0:6881->6881/tcp, 0.0.0.0:6881->6881/udp, 0.0.0.0:8080->8080/tcp   qbittorrent
# 设置重启方式
docker update --restart=always <你的容器ID，例如 2d1d22f00ddb>
```