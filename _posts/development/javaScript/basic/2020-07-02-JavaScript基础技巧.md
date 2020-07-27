---
title: "JavaScript 基础技巧"
subtitle: ""
layout: post
author: "luoruiqing"
header-style: text
tags:
  - JavaScript
---

## 基础类型

#### 对象与列表互转

- `Object.entries` : 对象转列表
- `Array.reduce` : 返回对象

```js
const source_obj = {"a": 1, "b":2}
Object.entries(source_obj).reduce((map, obj) => {  map[obj[0]] = obj[1];  ;return map },{})
```



<!-- arr.reduce(function(map, obj) {
    map[obj.key] = obj.val;
    return map;
}, {}); -->