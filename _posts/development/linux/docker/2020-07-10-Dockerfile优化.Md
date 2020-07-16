---
title: "Dockerfile 说明与技巧"
subtitle: ""
layout: post
author: "luoruiqing"
header-style: text
tags:
  - Docker
---


## 指令

#### 传递镜像版本

```docker
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER v1.0.0
RUN echo $CONT_IMG_VER
```

外部指定版本号, 若不存在则使用默认版本号

```sh
docker build --build-arg CONT_IMG_VER=v2.0.1 .
```

## 命令

#### build

`-f` : 用于指定Dockerfile文件

> docker build -f /path/to/a/Dockerfile 

`-t` : 指定镜像名称及版本

> docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .


## 优化

#### .dockerignore

> 在Dockerfile启动时, 会检测.dockerignore文件, 如果该文件存在, 则会忽略其中的文件以加速构建过程

匹配规则:

- `# comment` : 备注信息, 直接忽略
- `*/temp*` : 排除**任何目录**下以 **temp** 开头的**文件**及**文件夹**
- `*/*/temp*` : 排除**二级目录**下以 **temp** 开头的**文件**及**文件夹**
- `temp?` : 排除**根目录**下以 **temp** 开头末尾包含 **1** 个字符的文件及文件夹
- `!README*.md` : 排除时 **保留** 以 **README** 开头的 **.md** 文件

#### 指令

- RUN : 镜像内运行命令
    - 根据层级的概念, 如果有 `apt-get` 等容器命令应该写在文件的`头部`, 加速后续构建镜像的耗时
    - 使用 `&&` 来连接多条可以分层的命令, 例如:
    