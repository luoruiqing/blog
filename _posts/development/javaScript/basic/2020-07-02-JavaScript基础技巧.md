---
title: "JavaScript 基础技巧"
subtitle: "对象析构 | 字符处理 | 类型互转"
layout: post
author: "luoruiqing"
header-style: text
catalog:    true
tags:
  - JavaScript
---

## 数据转换


#### 展开对象所有属性

```js
// 可以考虑 console.table
function dir(obj){ for (let [key, value] of (obj || {})) console.log(key + ': \t' + value)}
```

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

## 数据操作


#### 树形操作

```js
export function combination_level(rows = [], father_key = 'father_id', key = 'children') {
    // 挂载子父关系
    const roots = []
    rows.forEach(row => {
        if (row[father_key]) { // 如果有父亲节点,寻找父亲节点
            let father = rows.filter(father => father.id == row[father_key]).map(father => father) // // 找爸爸,单个父亲的情况
            if (father.length) { // 有父亲
                father = father[0]
                father[key] = father[key] || [] // 创建孩子挂载的点
                father[key].push(row) // 将子菜单的引用追加到父亲节点上
            } else { // 没有父亲
                row[key] = [] // 空的孩子列表
                roots.push(row) // 其本身就是根
            }
        }
    })
    const base_roots = rows.filter(row => !row[father_key]).map(row => row) // 取出father_id为0或者undefined的树
    return roots.concat(base_roots) // 与基础根合并
}

export function digging(leaf, callback, level = 0, children_key = 'children', root = undefined) {
    // 下钻嵌套树形数据 本节点, 父亲节点
    let leafs // 当前节点的所有子节点
    if (Array.isArray(leaf)) { // 当前节点是列表
        leafs = leaf // 兼容类型, 子节点是第一个参数, 并没有父节点
    } else { // 非列表类型, 只有一个根节点
        level = level + 1
        leaf.root = root
        callback(leaf, root, level) // 调用, 每层等级加1
        leafs = leaf[children_key] || [] // 获取子节点
        root = leaf // 当前节点变成了子节点的根
    }
    leafs.forEach(leaf => digging(leaf, callback, level, children_key, root)) // 展开当前节点的子节点
}
```

#### 字符类型数字检测


```js
const NUMBER_REGEX = /^(-|)\d+(\.\d+)?$/ // 包含负数
export function isNumberStr(string) {
    return NUMBER_REGEX.test(string)
}
```

