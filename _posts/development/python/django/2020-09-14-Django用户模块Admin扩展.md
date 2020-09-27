---
title: "Django 用户模块Admin扩展"
subtitle: "Django Admin | User | 用户 | 拓展 | 扩展 | 继承"
layout: post
author: "luoruiqing"
header-style: text
tags:
  - Django

---


上篇说了如何扩展用户模块, 这篇说一下扩展后的用户模块怎么支持`admin`


#### 正常注册admin的方法


```py
# Accounts 应用
from django.contrib import admin
from .models import User


@admin.register(User)
class UserAdmin(admin):
    pass
```

界面:

![img]({{ site.baseurl }}/img/in-post/development/python/django/user-admin/01.png " ")


顺序错乱而且不够美观, 而且与admin默认的用户操作界面差异较大, 那我们可以考虑通过继承 `UserAdmin` 来解决


#### 查看源码

![img]({{ site.baseurl }}/img/in-post/development/python/django/user-admin/02.png " ")

看了下配置项, 还是挺多的, 想使用默认的功能, 就老实继承Admin然后扩展 `fieldsets` 吧


```py
from django.utils.translation import gettext_lazy as _
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as UserAdminBase
from .models import User


@admin.register(User)
class UserAdmin(UserAdminBase):
    fieldsets = UserAdminBase.fieldsets + (
        (_('用户信息'), {'fields': ('name_cn', 'name_en', 'phone', 'avatar_url', 'info')}),
    )

```

![img]({{ site.baseurl }}/img/in-post/development/python/django/user-admin/04.png " ")