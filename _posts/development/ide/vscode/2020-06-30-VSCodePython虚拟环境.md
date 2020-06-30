---
title: "VSCode Python手动配置虚拟环境"
subtitle: ""
layout: post
author: "luoruiqing"
header-style: text
tags:
  - IDE
  - VSCode
  - Python
---

关于搭建的教程很多, 这里说一下 Python 手动配置虚拟环境

## Python 选择解析器

#### 选择版本

![img]({{ site.baseurl }}/img/development/ide/vscode/python/vsocde-python1.png)

点击这里可以

![img]({{ site.baseurl }}/img/development/ide/vscode/python/vsocde-python2.png)

#### 选择手动

![img]({{ site.baseurl }}/img/development/ide/vscode/python/vsocde-python3.png)

这时会要求输入一个 Python 的路径, 以我的工程`project`为例:

```sh
cd ~/project # 进入工程目录
python -m venv ./.venv # 创建虚拟环境(此命令需要高Py版本才支持)
```

生成的路径是`project/.venv`文件夹, 所以虚拟环境所在目录应该是`/project/.venv`, 根据 VSCode 的预置变量组合为: `${workspaceFolder}/project/.venv`, 将该路径输入到环境内, 则完成切换.
