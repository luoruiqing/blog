---
title:        "Vue router & iframe 标签刷新问题"
subtitle:     "Vue router在切换后总是刷新iframe的解决办法"
layout:       post
author:       "luoruiqing"
header-style: text
catalog :     true
tags:
  - vue-router
  - vue
---



小伙伴们在开发一些业务系统的时候, 可能会遇到需要嵌入iframe标签的情况, 如果是单页Vue没有使用Vue-router组件时 不会出现问题, 一旦使用了路由组件, 并且在路由指定的组件内使用iframe就会发生切换路由, iframe总是被重复清除, 而不被缓存的问题.


### 例子

```vue
<template>
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
</template>
```

### 解决方式:

改动主要是几点:

- `router-view` 的同级创建`iframe`标签(不能在`router-view`或组件内)
- `iframe`增加`v-show`来分离`常规路由`和`外链路由`
- 定义路由时, 在外链的路由不设置`component`属性(或者设置空白组件)

**使用`router-view`的地方**
```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe v-show=" $route.path == '/menu/iframe' "></iframe>
</template>
```

**定义路由的地方**

```js
{
    // 根路由
    path: '/menu',
    component: Layout, // 布局, 此布局包含上面的router-view
    meta: {
        title: '菜单配置',
        icon: 'nested',
    },
    // 子路由
    children: [
        { // 外链组件
            path: 'iframe',
            name: 'IFrameView',
            // component 这里跳过, 保持页面空白
            meta: { title: '外链', icon: 'card' }
        },
        { // 常规组件
            path: 'editor',
            name: 'MenuEditorView',
            component: () => import('@pc/views/menu/edit/index.vue'),
            meta: { title: '菜单编辑', icon: 'nested' }
        }
    ]
}
```

此时页面脱离了vue-router的管控, 获得符合直觉的结果


### 多外链

多外链包含两种情况:

- 固定数量
- 动态数量


#### 固定数量

```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe src="/a"></iframe>
  <iframe src="/b"></iframe>
  <iframe src="/c"></iframe>
</template>
```

```js
{
    // 根路由
    path: '/menu',
    component: Layout, // 布局, 此布局包含上面的router-view
    meta: {
        title: '菜单配置',
        icon: 'nested',
    },
    // 子路由
    children: [
        { // 外链组件
            path: 'a',
            name: 'IFrameView1',
            meta: { title: '外链a', icon: 'card' }
        },
        { // 外链组件
            path: 'b',
            name: 'IFrameView2',
            meta: { title: '外链b', icon: 'card' }
        },
        { // 外链组件
            path: 'c',
            name: 'IFrameView3',
            meta: { title: '外链c', icon: 'card' }
        },
    ]
}
```

#### 动态数量

```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe v-for="" src="/a"></iframe>
  <iframe src="/b"></iframe>
  <iframe src="/c"></iframe>
</template>
```

## 查阅


## 查阅


## 查阅


### 查阅


### 查阅


### 查阅


## 查阅



作者：掉毛蛙
链接：https://www.jianshu.com/p/e5851918927c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

使用场景, 子组件`$emit`父组件`异步`方法, 父组件执行后, 子组件执行`剩余`的部分, 用起来大概像是这样

下列例子均以 父组件方法 - `fetchData` 子组件方法 - `fetch` 对应展示例子

```vue
<!-- 子组件 -->
<script>
  {
    async fetch() {
      this.loading = true
      await this.$emit('fetchData') // 异步父组件
      this.loading = false // 父组件完成后, 子组件剩余的事情
    }
  }
</script>
<!-- 父组件 -->
<script>
  {
    async fetchData() {
      this.data = await (axios.get('/')).data // 获取数据
    }
  }
</script>
```

--- 

## 解决方式
目前来说有以下几种方式可以实现.
1. `$emit`方式, 传入成功和失败的`回调`
2. `props`传入异步`function`对象
3. `vuex`的方式


### 一. `$emit`方式
因为官方不支持emit返回值的方式, 所以最简单的是通过传入回调执行
#### 传入成功和失败的`回调`
```vue
<!-- 子组件 -->
<script>
  {
    fetch() { // 这个方法也可以是async/await方法
      this.loading = true // 执行前
      this.$emit('fetchData', () => this.loading = false, console.error) // 完成后和失败的回调
    }
  }
</script>
<!-- 父组件 -->
<script>
  {
    async fetchData(resolve, reject) {
      try {
        this.data = await (axios.get('/')).data // 获取数据
        resolve() // 执行成功
      } catch (e) {
        reject(e) // 执行失败
      }
    }
  }
</script>
```

### 二. `props`方式

```vue
<!-- 子组件 -->
<script>
  {
    props: {
      fetchData: {
        type: Function,
        default() {
          return new Promise((resolve, reject) => {
            try { resolve() } catch (e) { reject() }
          })
        }
      }
    },
    methods: {
      async fetch() {
        this.loading = true // 执行前
        const data = await fetchData() // 直接父组件函数
        this.loading = false
      }
    }
  }
</script>
<!-- 父组件 -->
<script>
  {
    async fetchData() {
      const data = await (axios.get('/')).data // 获取数据
      return data
    }
  }
</script>
```


### 三. [`Vuex`](https://vuex.vuejs.org)的方式
这种方式使用`dispatch`即可, 这里不做赘述, 请自行了解
