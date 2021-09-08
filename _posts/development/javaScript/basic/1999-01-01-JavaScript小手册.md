---
title: "JavaScript 小手册"
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
function combination_level(rows = [], father_key = 'father_id', key = 'children') {
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

function digging(leaf, callback, level = 0, children_key = 'children', root = undefined) {
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
// 二维表格转树,300层回调限制
function toTree(array, ignores = [undefined, null], key = "children", vkey='value') {
  const SEPARATOR = '/-----/' // 分隔符
  const cache = {} // 缓存所有节点
  const roots = [] // 结果
  for (let [y, row] of array.entries()) { // 列走向
    for (let [x, item] of row.entries()) { // 行走向
      if (ignores.indexOf(item) > -1) break // 截断或忽略的数据
      const flag = row.slice(0, x + 1).join(SEPARATOR)// 通过所有层级唯一
      const current = cache[flag] = cache[flag] || { x, y, [vkey]: value: item, [key]: [], span: 1 } // 缓存信息，存在则不二次生成
      if (x == 0 && roots.indexOf(current) === -1) roots.push(current) // 根级别
      else {
        const parent = cache[row.slice(0, x).join(SEPARATOR)] // 父节点是否记录在缓存内
        // 父节点存在，子节点未加入到父节点的子节点内
        if (parent) {
          if (parent[key].indexOf(current) === -1) parent[key].push(current)
          if (y !== parent.y) parent.span += 1
        }
      }
    }
  }
  return roots
}

```


#### 行列转置

```js
function transpose(array) {
  const table = []
  for (let [y, row] of array.entries()) { // 列走向
    for (let [x, value] of row.entries()) { // 行走向
      table[x] = table[x] || []
      table[x][y] = value
    }
  }
  return table
}
```

#### 二维表格合并为单元格

```js
function mergeTable(array, direction = 'y', callback = str => typeof (str) === 'string') {
  // array 二维表格  direction 走向： x 水平合并， y垂直合并, callback 回调函数， 检测是否参与合并， 传入 （单元格值，y坐标， x坐标）， 默认只合并字符类型数据
  const table = []

  for (let y = 0; y < array.length; y++) {  // 列走向
    for (let x = 0; x < array[y].length; x++) { // 行走向
      const value = array[y][x] // 当前单元格值
      const row = table[y] = table[y] || [] // 初始化每行
      row[x] = { value, x, y, span: 1 } // 初始化当前单元格

      if (!callback(value, y, x)) continue // 不合并的数据忽略

      if (direction === 'x') { // 横向对比
        let last = x - 1
        while (last > -1 && array[y][last] === value) last-- // 寻找上一个不同的单元格值
        if (x !== last + 1) { // 确实有偏移
          table[y][last + 1].span += 1 // 同名单元格范围 +1
          row[x] = null // 清空当前单元格
        }
      }
      else if (direction === 'y') { // 纵向对比
        let last = y - 1
        while (last > -1 && array[last][x] === value) last--
        if (y !== last + 1) {
          table[last + 1][x].span += 1
          row[x] = null
        }
      }

    }
  }
  return table
}
```
#### 字符类型数字检测


```js
const NUMBER_REGEX = /^(-|)\d+(\.\d+)?$/ // 包含负数
export function isNumberStr(string) {
    return NUMBER_REGEX.test(string)
}
```


#### range

```js
// 生成长度100的列表
Array.from({ length: 100 }, (_, x) => x)
```

#### lodash
```js
_.isEmpty // 判断对象是否为空
_.includes(collection, value, [fromIndex=0]) // 值是否存在这个集合中 
_.omit(object, [props]) // 忽略属性

//  根据key列表转对象 
_.keyBy(collection, [iteratee=_.identity])
// => _.keyBy(array, 'dir') -> {'dir':{}}

// 超出截断字符
_.truncate('hi-diddly-ho there, neighborino', {
  'length': 24,
  'separator': /,? +/
});
// => 'hi-diddly-ho there...'

```