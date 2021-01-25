---
title: "Django graphene-django 使用教程"
subtitle: "Django | graphql | graphene-django"
layout: post
author: "luoruiqing"
header-style: text
catalog: true
tags:
  - Django

---


## 安装

```sh
pip install graphene-django
```


#### 配置

```py
# settings.py

INSTALLED_APPS = [
    # ...
    'graphene_django', # 添加组件
]
 
GRAPHENE = { 'SCHEMA': '路径.schema' } # 这里schema为文件内的变量, 对应 schema = graphene.Schema(...)  后面有说明

```

#### 路由

```py
# urls.py
from django.urls import path
from graphene_django.views import GraphQLView

from .schemas import schema


urlpatterns = [
    path('graphql/', GraphQLView.as_view(graphiql=True, schema=schema)),
]

```


#### schemas

```py
# schemas.py
import graphene

class Query(graphene.ObjectType):
    hello = graphene.String(default_value="Hi!")

schema = graphene.Schema(query=Query) # GRAPHENE['SCHEMA'] 配置对应该文件的地址和变量名

```


#### 验证

启动服务后访问: [http://localhost:8000/graphql/#query=%7B%0A%20%20hello%0A%7D%0A](http://localhost:8000/graphql/#query=%7B%0A%20%20hello%0A%7D%0A)

![img]({{ site.baseurl }}/img/in-post/development/python/django/graphene-django/p1.png "...")

#### CSRF 

去除CSRF保护(非必要配置)

```py
# urls.py

from django.urls import path
from django.views.decorators.csrf import csrf_exempt

from graphene_django.views import GraphQLView

urlpatterns = [
    # ...
    path("graphql", csrf_exempt(GraphQLView.as_view(graphiql=True))),
]
```

## 基础用法

#### 模型定义

```py
# cookbook/ingredients/models.py
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Ingredient(models.Model):
    name = models.CharField(max_length=100)
    notes = models.TextField()
    category = models.ForeignKey(
        Category, related_name="ingredients", on_delete=models.CASCADE
    )

    def __str__(self):
        return self.name
```

> 注意: 别忘记注册APP和迁移表结构

#### 查询定义

```py
# cookbook/schema.py
import graphene
from graphene_django import DjangoObjectType

from cookbook.ingredients.models import Category, Ingredient # 你的模型(表)

class CategoryType(DjangoObjectType):
    class Meta:
        model = Category
        fields = ("id", "name", "ingredients") # 可供查询的字段

class IngredientType(DjangoObjectType):
    class Meta:
        model = Ingredient
        fields = ("id", "name", "notes", "category")

class Query(graphene.ObjectType):
    all_ingredients = graphene.List(IngredientType)
    category_by_name = graphene.Field(CategoryType, name=graphene.String(required=True))

    def resolve_all_ingredients(root, info):
        # We can easily optimize query count in the resolve method
        return Ingredient.objects.select_related("category").all()

    def resolve_category_by_name(root, info, name):
        try:
            return Category.objects.get(name=name)
        except Category.DoesNotExist:
            return None

schema = graphene.Schema(query=Query)
```

> 这里有个规则是 `Query.节点` 则需要创建方法 `resolve_节点` 的方法来编写执行过程, 这一点与 graphene 的设定一致.


#### 普通查询

```graphql
query {
  allIngredients {
    id
    name
  }
}
```


结果:

```json
{
  "data": {
    "allIngredients": [
      {
        "id": "1",
        "name": "Eggs"
      },
      {
        "id": "2",
        "name": "Milk"
      },
      {
        "id": "3",
        "name": "Beef"
      },
      {
        "id": "4",
        "name": "Chicken"
      }
    ]
  }
}
```


#### 条件查询

```graphql
query {
  categoryByName(name: "Dairy") {
    id
    name
    ingredients {
      id
      name
    }
  }
}
```

结果:

```json
{
  "data": {
    "categoryByName": {
      "id": "1",
      "name": "Dairy",
      "ingredients": [
        {
          "id": "1",
          "name": "Eggs"
        },
        {
          "id": "2",
          "name": "Milk"
        }
      ]
    }
  }
}
```

这时可以发现 `graphene-django` 模块帮我们完成了 Django 模型与 graphene 的信息对接, 使得查询配置变得极其简单快捷.

## 进阶

进阶教程请直接参考[文档](https://docs.graphene-python.org/projects/django/en/latest/).

## 常见错误

#### 转换失败


```log
Exception: Don't know how to convert the Django field demo.TestModel1.tags (<class 'taggit.managers.TaggableManager'>)
```

这个错误是模型定义时候使用了特殊的字段, 导致的字段类型未被识别而引发的错误, 首先可以创建一个 **标量**, 用于处理字段的转换过程, 然后注册一个转换函数说明数据关系和类型.

```py
import graphene
from 模型文件 import 模型


class CustomScalar(graphene.Scalar):
    ''' 自定义标量 '''
    @classmethod
    def serialize(cls, value):
        ''' 序列化, 转换过程, 最终需要一个可识别的标量对象 '''
        import json
        result = json.loads(value.value) # 转换
        return result


@convert_django_field.register(模型.无法识别的字段)
def convert_custom(field, registry=None):
    return graphene.Field(CustomScalar, description=field.help_text, required=not field.null)

```

#### django-taggit

该模块的实现的是特殊的外键 **TaggableManager** , 也会导致的字段的类型未被识别而引发的错误 Exception: Don't know how to convert the Django field...

这是我的模型:

```py
from django.db import models
from taggit.managers import TaggableManager


class TestModel1(models.Model):
    """ 测试表 """
    name = models.CharField(max_length=255, help_text="名称")
    tags = TaggableManager()

```

首先我们知道 `DjangoObjectType` 是模型的基本节点, `DjangoListField` 是模型列表的基本节点, 而 TestModel1.tags 与 DjangoListField 的 API 一致, 根据 **graphene_django** 提供的转换注册方法, 直接对应好关系即可.

```py
from taggit.managers import TaggableManager
from taggit.models import Tag
from graphene_django.converter import convert_django_field

# 给 taggit.Tag 模型添加一个 DjangoObjectType
class TaggitType(graphene_django.DjangoObjectType):
    class Meta:
        model = taggit.models.Tag

@convert_django_field.register(TaggableManager) # 指定模型中未识别的对象
def convert_taggit(field, registry=None):
     # 多对多的关系, 注册为 DjangoListField 字段, 类型指定为 TaggitType
     return graphene_django.DjangoListField(TaggitType, description=field.help_text, required=not field.null) 
```


## 注意

#### 驼峰转换

`auto_camelcase`(True): 表示是否开启驼峰转换, 不开启需设置为 **False**

```py
# schemas.py
import graphene

class Query(graphene.ObjectType):
    hello = graphene.String(default_value="Hi!")

schema = graphene.Schema(query=Query, auto_camelcase=False) # 在生成 Schema 的时候设置

```


