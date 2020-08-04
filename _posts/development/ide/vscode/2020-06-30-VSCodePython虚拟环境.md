---
title: "VSCode Python虚拟环境"
subtitle: "VSCode 手动配置Python虚拟环境"
layout: post
author: "luoruiqing"
header-style: text
tags:
  - VSCode
  - Python
---

关于搭建的教程很多, 这里说一下 `Python` 在 `VSCode` 中如何手动配置 **`虚拟环境`**, 需要提前安装好 VSCode 的 Python 插件. 以我的工程`project`为例:

#### 生成虚拟环境

```sh
cd ~/project # 进入工程目录
python -m venv ./.venv # 创建虚拟环境(此命令需要高Py版本才支持)
# source .venv/bin/activate # 激活虚拟环境
```

此时在`project`目录下会生成 `.venv` 的文件夹, **`相对路径`** 目录为 `/project/.venv`

## Python - 选择解析器

#### 选择版本

一种是通过 `command + shift + P` 调出命令输入框

![img]({{ site.baseurl }}/img/in-post/development/ide/vscode/python/vsocde-python1.png)

或者点击左下角状态栏

![img]({{ site.baseurl }}/img/in-post/development/ide/vscode/python/vsocde-python2.png)

#### 选择手动

![img]({{ site.baseurl }}/img/in-post/development/ide/vscode/python/vsocde-python3.png)

根据 VSCode 的`预置变量`与`虚拟环境` 组合路径为: `${workspaceFolder}/project/.venv`, 将该路径输入到环境内.

> `${workspaceFolder}`: 在 VSCode 中打开的文件夹的路径


![img]({{ site.baseurl }}/img/in-post/development/ide/vscode/python/vsocde-python4.png)

此时发现左下角的环境已经发生切换, 再配置调试即可.

