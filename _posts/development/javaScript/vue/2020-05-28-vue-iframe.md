---
title:        "Vue router & iframe 标签刷新问题"
subtitle:     "Vue router在切换后总是刷新iframe的解决办法"
layout:       post
author:       "luoruiqing"
header-style: text
catalog :     true
tags:
  - Vue
---



小伙伴们在开发一些业务系统的时候, 可能会遇到需要嵌入`iframe`标签的情况, 如果是单页`Vue`没有使用`Vue-router`组件时 不会出现问题, 一旦使用了路由组件, 并且在路由指定的组件内使用`iframe`就会发生切换路由, iframe总是被`重复清除`, 而不被`缓存`的问题.


### 路由例子

```vue
<template>
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
</template>
```

### 解决方式

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

此时页面脱离了vue-router的管控, 获得符合直觉的结果, 标签页面也可以正常展示


### 多外链

多外链包含两种情况:

- 固定数量
- 动态数量


#### 固定数量

通过v-show来展示不同的iframe, 但是要有所区分

```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe v-show=" $route.path == '/menu/a' " src="/a"></iframe>
  <iframe v-show=" $route.path == '/menu/b' " src="/b"></iframe>
  <iframe v-show=" $route.path == '/menu/c' " src="/c"></iframe>
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

如果你使用了类似`标签页`的设计, 不同的`iframe`希望单独一个标签页的时候. 这里的`name`属性需要设置为不同的name名称, 以保证标签页的标题不会出现问题 


#### 动态数量

有两种方式:
- 不添加路由
- 动态装填路由(查阅vue-router教程)

不管是哪种, 模板的写法都如下即可:

```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe v-for="i in iframes" :key="i.src" v-show=" current_iframe == i.name " :src="i.src"></iframe>
</template>
```

这种方式可以实现循环多个iframe页面的展示, 保证`name`不同, 装填到路由内. 如果有类似`标签页`的设计, 务必动态添加路由.

### 标签页

如果使用了`标签页`的设计, 希望在标签关闭的同时关闭iframe, 则需要使用`v-if`判定什么时候关闭元素, 还是以单个外链为例:


```vue
<template>
  <!-- 常规路由 -->
  <keep-alive :include="cachedViews">
      <router-view :key="key" />
  </keep-alive>
  <!-- 内嵌外链 -->
  <iframe v-if=" cachedViews.indexOf('MenuEditorView') > -1 "
          v-show=" $route.path == '/menu/iframe' "
  ></iframe>
</template>
```

`v-if`的判断如果是缓存中不存在这个组件, 即销毁元素, `cachedViews`和组件的`name`是实现该功能的关键, 如果你是个Vue的老手应该明白怎么做了吧.

以上都是简单的实例, 可以根据业务场景进行改动和封装.




