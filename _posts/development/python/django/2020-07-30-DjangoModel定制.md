---
title: "Django django-reversion详解"
subtitle: "Django Model历史修改记录及删除回滚"
layout: post
author: "luoruiqing"
catalog: true
header-style: text
tags:
  - Django
---

# [django-reversion](https://django-reversion.readthedocs.io/)

> django-reversion is an extension to the Django web framework that provides version control for model instances.


## 安装过程

- 安装模块 `django-reversion`.
- 添加 `'reversion'` 到 `settings.py` 配置文件中 `INSTALLED_APPS` 的选项内
- 运行迁移 `manage.py migrate`.


**注意: 每当注册新的Model时, 运行一次`createinitialrevisions`**


## 快速开始

#### 模型注册

```python
from django.db import models
import reversion

@reversion.register()
class YourModel(models.Model):

    pass
```

#### Admin注册

```python
from django.contrib import admin
from reversion.admin import VersionAdmin

@admin.register(YourModel)
class YourModelAdmin(VersionAdmin):

    pass
```

## 高级用法

#### 创建历史

...

#### 如何回滚

...

## API一览

...

## 注意事项

- 注意: 每当注册新的Model时, 运行一次`createinitialrevisions`
- `Queryset.update()` 方法不会生成历史记录