---
title: "Python3 处理空白HTML占位空白字符"
subtitle: "Python | 空白字符处理 | u3000 | x0a "
layout: post
author: "luoruiqing"
header-style: text
tags:
  - Python
  - 爬虫
---


## unicodedata

Python自带一个 unicodedata 的包, 方便部分字符中携带一些在网页中不可见的占位空白字符, 可以通过这个包进行过滤, 示例如下:


```python
import unicodedata


ucd.normalize('NFKC', '\n|\r|\t|\u3000|\xa0|\u2003')
# '\n|\r|\t| | | '

```

