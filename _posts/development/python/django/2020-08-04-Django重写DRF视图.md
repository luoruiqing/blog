---
title: "Django 定制DRF视图敏捷开发"
subtitle: ""
layout: post
author: "luoruiqing"
catalog: true
header-style: text
tags:
  - Django
---



## 定制 

定制`DRF`目的是用于`敏捷开发`, 主要包含的功能如下:

- 请求前进行参数兼容
- `断言`抛出或`自定义错误`的正确识别并响应为`400`错误
- 业务视图直接返回`Py基本类型`即可封装成`django HttpResponse` 对象 (JSON响应)


完成上述工作需要做以下几个模块:

- 扩展 `Exception` : 用于自定义错误
- 扩展 `django mtd` 方法: 用于兼容个性化类型及django类型
- 扩展 `rest_framework.views.APIView` 视图: 正确执行流程并捕获错误
- 使用

#### 扩展 `Exception`


**error.py**

```py

```


```py
from collections import Iterable
from django.db.models import Model, QuerySet
from django.forms.models import model_to_dict as django_mtd


def model_to_dict(instance, fields=None, exclude=None):
    result = {}  # 原类型
    if isinstance(instance, dict):  # 字典类型
        for key, value in instance.items():
            if fields and key not in fields:  # 保留指定字段
                del instance[key]  # 清除
                continue
            if exclude and key in exclude:  # 排除指定字段
                del instance[key]
                continue
            result[key] = model_to_dict(value, fields=fields, exclude=exclude)  # 回调更新
    elif isinstance(instance, (list, Iterable, QuerySet)) and not isinstance(instance, str):  # 除字符类型的任何可迭代对象包含生成器
        result = [model_to_dict(row, fields=fields, exclude=exclude) for row in instance]  # 回调更新
    elif isinstance(instance, Model):
        result = django_mtd(instance, fields=fields, exclude=exclude)
    return result or instance


mtd = model_to_dict
```

#### 视图基类

```py
import functools
from collections import Iterable

from django.conf import settings
from django.core.exceptions import ObjectDoesNotExist
from django.db.models import Model, QuerySet
from django.http.response import (HttpResponse, HttpResponseBase,
                                  StreamingHttpResponse)

from rest_framework.response import Response as RestfulResponse
from rest_framework.views import APIView as RestFrameworkAPIView
from utils.base import model_to_dict

from ..error import OBJECT_DOES_NOT_EXIST, ResponseExceptionBase

RESPONSE_JSON = {"code": 200, 'message': "完成", "description": "success", "data": {}}


class CustomAPIViewBase(RestFrameworkAPIView):
    '''  基础视图 '''


class CustomAPIView(CustomAPIViewBase):
    """ 自定义流程视图 """

    # authentication_classes = (WithoutCsrfValidationSessionAuthentication, BasicAuthentication)
    METHODS = ['get', 'post', 'put', 'delete']

    def __init_subclass__(cls, **kwargs):
        methods = [(method, getattr(cls, method)) for method in cls.METHODS if hasattr(cls, method)]
        if methods:  # 装饰方法
            [setattr(cls, method, cls.__custom_process(func)) for (method, func) in methods]
        return super().__init_subclass__(**kwargs)

    @staticmethod
    def __custom_process(func):
        ''' 装饰器实现错误收集和数据类型转换 '''
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            try:
                # 请求前 ******************************************************************
                request = args[1]  # 请求对象
                IS_JSON_TYPE = 'application/json' in request.META.get('HTTP_ACCEPT', '')
                try:  # 增加JSON
                    request.json = loads(body) if IS_JSON_TYPE and body else {}
                except:  # 提交json错误
                    raise REQUEST_JSON_ERROR
                # 处理 ********************************************************************
                response = func(*args, **kwargs)
                # 响应后 ******************************************************************
                if isinstance(response, (ResponseExceptionBase,)):  # 自定义错误返回
                    raise response  # 交给 process_exception 处理
                elif isinstance(response, str):
                    response = HttpResponse(response)  # 字符类型的返回
                elif not isinstance(response, HttpResponseBase):  # 如果不是响应对象
                    if isinstance(response, (QuerySet, Model, Iterable)):  # 模型对象/查询结果对象/迭代对象(字典|列表|集合等) 转字典
                        response = model_to_dict(response)
                    response = RestfulResponse({**RESPONSE_JSON.copy(), **{'data': response}})  # 其他类型的返回
                return response
            except Exception as error:
                # 错误处理 ******************************************************************
                if isinstance(error, AssertionError) and error.args:  # 通过断言抛出的已知错误
                    error_content = error.args[0]
                    if isinstance(error_content, ResponseExceptionBase):
                        error = error_content  # 实例则覆盖错误变量
                elif isinstance(error, ObjectDoesNotExist):  # 模型根据id get方法报错整体处理
                    error = OBJECT_DOES_NOT_EXIST(error.args[0].split(" ", 1)[0])
                # 正式处理已知错误
                if isinstance(error, AssertionError):
                    message = str(error)
                    return RestfulResponse({"status": 400, 'message': message, "description": message, "data": {}}, status=400)  # 默认400错误
                elif isinstance(error, ResponseExceptionBase):
                    return ExceptResponse(error)  # 默认400错误
                else:  # 线上不暴露错误 但依旧是JSON返回
                    if settings.DEBUG:  # 所有错误直接抛出
                        raise error
                    logger.error(format_exc())  # 输出错误信息
                    error = str(error)
                    return RestfulResponse({"status": 500, 'message': f'服务器错误.{error}', "description": error, "data": {}, }, status=500)  # 状态码为500
                # 错误处理
        return wrapper


class APIView(CustomAPIView):
    ''' 视图 '''

```


## 使用