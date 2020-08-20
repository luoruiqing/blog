---
title: "JavaScript 基础技巧"
subtitle: ""
layout: post
author: "luoruiqing"
header-style: text
catalog:    true
tags:
  - JavaScript
---

## 类型转换

#### 对象转列表

```js
let entries = [
    ['1', '张三'],
    ['2', '李四']
]

let obj = Object.fromEntries(entries)

console.log(JSON.stringify(obj))
// {1: "张三", 2: "李四"}
```

#### 列表转对象

```js
let list = [
    {
        id: 1,
        name: '张三',
    },
    {
        id: 2,
        name: '李四',
    },
]
let result = list.reduce((obj, item) => {
    obj[item.id] = item // 设置属性
    return obj
}, {})

console.log(JSON.stringify(result))
// {"1":{"id":1,"name":"张三"},"2":{"id":2,"name":"李四"}}
```

